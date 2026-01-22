# CI/CD con GitHub Actions

> **Automatizar testing, build y deployment**

---

## ¿Qué es CI/CD?

**CI (Continuous Integration)** = Integrar código frecuentemente y ejecutar tests automáticamente

**CD (Continuous Deployment)** = Desplegar automáticamente a producción

**Flujo típico:**
```
Push a GitHub → Tests automáticos → Build → Deploy a producción
```

---

## GitHub Actions

**GitHub Actions** = CI/CD integrado en GitHub

**Conceptos:**
- **Workflow:** Proceso automatizado
- **Job:** Conjunto de pasos
- **Step:** Acción individual
- **Runner:** Máquina que ejecuta el workflow

---

## Workflow básico

```.github/workflows/test.yml
name: Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test-backend:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          cd backend
          pip install -r requirements.txt
      
      - name: Run tests
        run: |
          cd backend
          pytest --cov
```

---

## Workflow completo: Backend

```.github/workflows/backend.yml
name: Backend CI/CD

on:
  push:
    branches: [ main ]
    paths:
      - 'backend/**'
  pull_request:
    branches: [ main ]
    paths:
      - 'backend/**'

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: test_db
          POSTGRES_USER: test_user
          POSTGRES_PASSWORD: test_password
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python 3.11
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
      
      - name: Install dependencies
        run: |
          cd backend
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest-cov
      
      - name: Run linting
        run: |
          cd backend
          pip install flake8
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
      
      - name: Run tests with coverage
        env:
          DATABASE_URL: postgresql://test_user:test_password@localhost:5432/test_db
          SECRET_KEY: test-secret-key
        run: |
          cd backend
          pytest --cov=. --cov-report=xml
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./backend/coverage.xml
          flags: backend
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Railway
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
        run: |
          npm install -g @railway/cli
          railway up
```

---

## Workflow completo: Frontend

```.github/workflows/frontend.yml
name: Frontend CI/CD

on:
  push:
    branches: [ main ]
    paths:
      - 'frontend/**'
  pull_request:
    branches: [ main ]
    paths:
      - 'frontend/**'

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node 18
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        run: |
          cd frontend
          npm ci
      
      - name: Run linting
        run: |
          cd frontend
          npm run lint
      
      - name: Run tests
        run: |
          cd frontend
          npm run test -- --coverage
      
      - name: Build
        run: |
          cd frontend
          npm run build
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./frontend/coverage/coverage-final.json
          flags: frontend
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          working-directory: ./frontend
```

---

## E2E Tests en CI

```.github/workflows/e2e.yml
name: E2E Tests

on:
  pull_request:
    branches: [ main ]

jobs:
  e2e:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: e2e_db
          POSTGRES_USER: e2e_user
          POSTGRES_PASSWORD: e2e_password
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v3
      
      # Setup Backend
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Start backend
        env:
          DATABASE_URL: postgresql://e2e_user:e2e_password@localhost:5432/e2e_db
        run: |
          cd backend
          pip install -r requirements.txt
          python manage.py migrate
          python manage.py runserver &
          sleep 10
      
      # Setup Frontend
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Start frontend
        run: |
          cd frontend
          npm ci
          npm run dev &
          sleep 10
      
      # Run E2E
      - name: Install Playwright
        run: |
          cd frontend
          npx playwright install --with-deps
      
      - name: Run Playwright tests
        run: |
          cd frontend
          npx playwright test
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: frontend/playwright-report/
```

---

## Secrets en GitHub

### Configurar secrets

1. GitHub → Settings → Secrets and variables → Actions
2. New repository secret
3. Agregar:
   - `RAILWAY_TOKEN`
   - `VERCEL_TOKEN`
   - `VERCEL_ORG_ID`
   - `VERCEL_PROJECT_ID`
   - `DATABASE_URL` (producción)
   - `SECRET_KEY`

### Usar secrets en workflow

```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
    run: |
      echo "Deploying with secret credentials"
```

---

## Matrix strategy (tests en múltiples versiones)

```yaml
name: Matrix Tests

on: [push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: ['3.10', '3.11', '3.12']
        node-version: ['16', '18', '20']
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Setup Node ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      
      - name: Run tests
        run: |
          echo "Testing on ${{ matrix.os }} with Python ${{ matrix.python-version }} and Node ${{ matrix.node-version }}"
```

---

## Conditional jobs

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: pytest
  
  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: echo "Deploying to staging"
  
  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: echo "Deploying to production"
```

---

## Artifacts (guardar resultados)

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build frontend
        run: |
          cd frontend
          npm ci
          npm run build
      
      - name: Upload build
        uses: actions/upload-artifact@v3
        with:
          name: frontend-build
          path: frontend/dist/
          retention-days: 7
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build
        uses: actions/download-artifact@v3
        with:
          name: frontend-build
          path: dist/
      
      - name: Deploy
        run: echo "Deploying dist/ to production"
```

---

## Caching (acelerar workflows)

### Cache de dependencias Python

```yaml
- name: Cache pip dependencies
  uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

### Cache de node_modules

```yaml
- name: Cache node_modules
  uses: actions/cache@v3
  with:
    path: frontend/node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

---

## Docker en CI/CD

```yaml
name: Docker CI/CD

on:
  push:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push backend
        uses: docker/build-push-action@v4
        with:
          context: ./backend
          push: true
          tags: user/backend:latest,user/backend:${{ github.sha }}
      
      - name: Build and push frontend
        uses: docker/build-push-action@v4
        with:
          context: ./frontend
          push: true
          tags: user/frontend:latest,user/frontend:${{ github.sha }}
```

---

## Notificaciones

### Slack

```yaml
- name: Notify Slack
  if: always()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Deployment ${{ job.status }}'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Email

```yaml
- name: Send email
  if: failure()
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 465
    username: ${{ secrets.EMAIL_USERNAME }}
    password: ${{ secrets.EMAIL_PASSWORD }}
    subject: CI/CD Failed
    to: admin@example.com
    from: GitHub Actions
    body: Build failed on ${{ github.repository }}
```

---

## Branch protection rules

1. GitHub → Settings → Branches
2. Add rule para `main`
3. Configurar:
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - ✅ Require conversation resolution before merging

**Esto previene:**
- Pushes directos a main
- Merges sin tests pasando
- Merges con conflictos

---

## Scheduled workflows (cron)

```yaml
name: Nightly Tests

on:
  schedule:
    # Ejecutar a las 2am UTC todos los días
    - cron: '0 2 * * *'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run comprehensive tests
        run: pytest --slow
```

---

## Workflow completo de producción

```.github/workflows/production.yml
name: Production Deployment

on:
  push:
    branches: [ main ]

jobs:
  # Tests
  test-backend:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: test_db
          POSTGRES_USER: test_user
          POSTGRES_PASSWORD: test_password
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: cd backend && pip install -r requirements.txt
      - run: cd backend && pytest --cov
  
  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: cd frontend && npm ci
      - run: cd frontend && npm run test -- --coverage
  
  # Build
  build-frontend:
    needs: [test-backend, test-frontend]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: cd frontend && npm ci
      - run: cd frontend && npm run build
      - uses: actions/upload-artifact@v3
        with:
          name: frontend-build
          path: frontend/dist/
  
  # Deploy
  deploy-backend:
    needs: [test-backend, test-frontend]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Railway
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
        run: railway up --service backend
  
  deploy-frontend:
    needs: build-frontend
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: frontend-build
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
  
  # Notify
  notify:
    needs: [deploy-backend, deploy-frontend]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment to production ${{ job.status }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Debugging workflows

### Logs

```bash
# Ver logs en GitHub:
# Actions tab → Click en workflow → Click en job → Ver steps
```

### Debugging con tmate

```yaml
- name: Setup tmate session
  uses: mxschmitt/action-tmate@v3
  if: failure()
```

### Verbose output

```yaml
- name: Run tests
  run: pytest -vv
```

---

## Resumen

**CI/CD** = Automatizar tests + build + deployment

**GitHub Actions** = CI/CD integrado en GitHub

**Workflow típico:**
1. **Push** a GitHub
2. **Tests** (backend + frontend)
3. **Build** (compilar frontend)
4. **Deploy** (Railway + Vercel)
5. **Notify** (Slack/Email)

**Beneficios:**
- ✅ Detectar bugs automáticamente
- ✅ Deploy seguro (solo si tests pasan)
- ✅ Consistency (mismo proceso siempre)
- ✅ Rapidez (automatización completa)

**Próximo módulo:** Monitoring y Logs

---

## Recursos

- **[GitHub Actions](https://docs.github.com/en/actions)** - Docs oficiales
- **[Awesome Actions](https://github.com/sdras/awesome-actions)** - Lista de actions útiles
- **[GitHub Actions Marketplace](https://github.com/marketplace?type=actions)** - Marketplace

¡CI/CD completo! 🚀
