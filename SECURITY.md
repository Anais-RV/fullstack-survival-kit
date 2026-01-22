# Nota de Seguridad

Este repositorio contiene **material educativo** sobre desarrollo fullstack.

## Sobre las "credenciales" detectadas

GitGuardian puede detectar patrones que parecen credenciales en este repositorio. Todas son **ejemplos educativos** en archivos de documentación Markdown (`.md`):

- ❌ No hay credenciales reales
- ❌ No hay secrets de producción
- ❌ No hay contraseñas reales

Las "credenciales" son **ejemplos didácticos** como:
- `password123` (contraseña de ejemplo en tutoriales)
- `test_password` (contraseña de test en CI/CD)
- `secret123` (secret de ejemplo en Docker)
- `tu-email@gmail.com` (placeholder en configuraciones)

## ¿Por qué están ahí?

Este material enseña conceptos de seguridad, autenticación, variables de entorno y buenas prácticas. Los ejemplos incluyen:

- Cómo **NO** guardar contraseñas (para luego enseñar cómo hacerlo bien)
- Configuración de variables de entorno
- Testing con credenciales de prueba
- Ejemplos de configuración de CI/CD

Todos los ejemplos están en contextos educativos claros, nunca como código de producción.

## Configuración

He añadido `.gitguardian.yaml` para indicar a GitGuardian que ignore los archivos `.md` (documentación educativa).

## ¿Eres estudiante usando este material?

**NUNCA** uses las contraseñas de ejemplo en tus proyectos reales. Este material enseña precisamente a:
1. Usar variables de entorno (`.env`)
2. No commitear secrets reales
3. Usar servicios de gestión de secrets
4. Configurar `.gitignore` correctamente

---

**Si tienes dudas sobre seguridad en tu propio proyecto, revisa:**
- [03-backend/bloque-06-arquitectura/33-variables-entorno.md](03-backend/bloque-06-arquitectura/33-variables-entorno.md)
- [08-despliegue/](08-despliegue/)
