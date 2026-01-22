# Upload de Archivos

> **Subir imágenes y archivos con React + Django + Cloudinary**

---

## Estrategias de almacenamiento

### Opción 1: Servidor local (desarrollo)
- ✅ Simple
- ❌ No escalable
- ❌ Problemas en deployment

### Opción 2: Cloud storage (producción)
- ✅ Escalable
- ✅ CDN integrado
- ✅ Transformaciones automáticas
- Opciones: **Cloudinary**, AWS S3, Google Cloud Storage

**Recomendación:** Cloudinary (tier gratuito generoso)

---

## Backend: Django con archivos locales

### 1. Configurar settings.py

```python
# core/settings.py
import os

# Media files (uploads)
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

### 2. URLs para servir archivos (desarrollo)

```python
# core/urls.py
from django.conf import settings
from django.conf.urls.static import static
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('productos.urls')),
]

# Servir archivos media en desarrollo
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### 3. Modelo con ImageField

```python
# productos/models.py
from django.db import models

class Producto(models.Model):
    nombre = models.CharField(max_length=200)
    imagen = models.ImageField(upload_to='productos/', blank=True, null=True)
    # ...
```

**Migrar:**
```bash
pip install Pillow  # Requerido para ImageField
python manage.py makemigrations
python manage.py migrate
```

### 4. Serializer

```python
# productos/serializers.py
from rest_framework import serializers
from .models import Producto

class ProductoSerializer(serializers.ModelSerializer):
    imagen_url = serializers.SerializerMethodField()
    
    class Meta:
        model = Producto
        fields = ['id', 'nombre', 'imagen', 'imagen_url', ...]
    
    def get_imagen_url(self, obj):
        if obj.imagen:
            request = self.context.get('request')
            if request:
                return request.build_absolute_uri(obj.imagen.url)
        return None
```

---

## Frontend: React con FormData

### 1. Input de archivo

```jsx
// src/components/ProductoForm.jsx
import { useState } from 'react';
import { productoService } from '../services/productoService';

export default function ProductoForm() {
    const [formData, setFormData] = useState({
        nombre: '',
        precio: '',
        imagen: null,
    });
    const [preview, setPreview] = useState(null);
    
    const handleFileChange = (e) => {
        const file = e.target.files[0];
        if (file) {
            setFormData({ ...formData, imagen: file });
            
            // Preview
            const reader = new FileReader();
            reader.onloadend = () => {
                setPreview(reader.result);
            };
            reader.readAsDataURL(file);
        }
    };
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        
        // Crear FormData
        const data = new FormData();
        data.append('nombre', formData.nombre);
        data.append('precio', formData.precio);
        if (formData.imagen) {
            data.append('imagen', formData.imagen);
        }
        
        const result = await productoService.create(data);
        
        if (result.success) {
            alert('Producto creado');
        } else {
            alert('Error');
        }
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <div>
                <label>Nombre:</label>
                <input
                    type="text"
                    value={formData.nombre}
                    onChange={(e) => setFormData({ ...formData, nombre: e.target.value })}
                    required
                />
            </div>
            
            <div>
                <label>Precio:</label>
                <input
                    type="number"
                    value={formData.precio}
                    onChange={(e) => setFormData({ ...formData, precio: e.target.value })}
                    required
                />
            </div>
            
            <div>
                <label>Imagen:</label>
                <input
                    type="file"
                    accept="image/*"
                    onChange={handleFileChange}
                />
                {preview && (
                    <img src={preview} alt="Preview" style={{ maxWidth: 200, marginTop: 10 }} />
                )}
            </div>
            
            <button type="submit">Crear Producto</button>
        </form>
    );
}
```

### 2. Servicio con FormData

```javascript
// src/services/productoService.js
import API from './api';

export const productoService = {
    create: async (data) => {
        try {
            const response = await API.post('/productos/', data, {
                headers: {
                    'Content-Type': 'multipart/form-data',
                },
            });
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
    
    update: async (id, data) => {
        try {
            const response = await API.put(`/productos/${id}/`, data, {
                headers: {
                    'Content-Type': 'multipart/form-data',
                },
            });
            return { success: true, data: response.data };
        } catch (error) {
            return { success: false, error: error.response?.data };
        }
    },
};
```

---

## Cloudinary: Almacenamiento en la nube

### 1. Crear cuenta

1. Ir a [cloudinary.com](https://cloudinary.com/)
2. Registrarse (gratis hasta 25 GB)
3. Obtener credenciales del dashboard

### 2. Instalar en Django

```bash
pip install cloudinary django-cloudinary-storage
```

### 3. Configurar Django

```python
# core/settings.py
import cloudinary
import cloudinary.uploader
import cloudinary.api

INSTALLED_APPS = [
    # ...
    'cloudinary_storage',
    'cloudinary',
    # ...
]

# Cloudinary
CLOUDINARY_STORAGE = {
    'CLOUD_NAME': 'tu-cloud-name',
    'API_KEY': 'tu-api-key',
    'API_SECRET': 'tu-api-secret',
}

# Media storage
DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'
```

**Con variables de entorno (.env):**
```python
from decouple import config

CLOUDINARY_STORAGE = {
    'CLOUD_NAME': config('CLOUDINARY_CLOUD_NAME'),
    'API_KEY': config('CLOUDINARY_API_KEY'),
    'API_SECRET': config('CLOUDINARY_API_SECRET'),
}
```

### 4. Modelo (igual que antes)

```python
# productos/models.py
class Producto(models.Model):
    imagen = models.ImageField(upload_to='productos/', blank=True, null=True)
```

**Cloudinary maneja el upload automáticamente.**

---

## Frontend: Upload directo a Cloudinary

### 1. Configurar Cloudinary en React

```env
# .env
VITE_CLOUDINARY_CLOUD_NAME=tu-cloud-name
VITE_CLOUDINARY_UPLOAD_PRESET=tu-preset
```

**Crear upload preset:**
1. Cloudinary Dashboard → Settings → Upload
2. Scroll a "Upload presets"
3. Add upload preset → Unsigned
4. Guardar el nombre del preset

### 2. Servicio de upload

```javascript
// src/services/uploadService.js
export const uploadService = {
    uploadImage: async (file, onProgress) => {
        const cloudName = import.meta.env.VITE_CLOUDINARY_CLOUD_NAME;
        const uploadPreset = import.meta.env.VITE_CLOUDINARY_UPLOAD_PRESET;
        
        const formData = new FormData();
        formData.append('file', file);
        formData.append('upload_preset', uploadPreset);
        
        try {
            const response = await fetch(
                `https://api.cloudinary.com/v1_1/${cloudName}/image/upload`,
                {
                    method: 'POST',
                    body: formData,
                }
            );
            
            if (!response.ok) {
                throw new Error('Error al subir imagen');
            }
            
            const data = await response.json();
            return {
                success: true,
                url: data.secure_url,
                publicId: data.public_id,
            };
        } catch (error) {
            return {
                success: false,
                error: error.message,
            };
        }
    },
};
```

### 3. Componente con upload

```jsx
// src/components/ProductoForm.jsx
import { useState } from 'react';
import { uploadService } from '../services/uploadService';
import { productoService } from '../services/productoService';

export default function ProductoForm() {
    const [formData, setFormData] = useState({
        nombre: '',
        precio: '',
        imagen: '',
    });
    const [uploading, setUploading] = useState(false);
    const [preview, setPreview] = useState(null);
    
    const handleFileChange = async (e) => {
        const file = e.target.files[0];
        if (!file) return;
        
        // Preview local
        const reader = new FileReader();
        reader.onloadend = () => {
            setPreview(reader.result);
        };
        reader.readAsDataURL(file);
        
        // Upload a Cloudinary
        setUploading(true);
        const result = await uploadService.uploadImage(file);
        setUploading(false);
        
        if (result.success) {
            setFormData({ ...formData, imagen: result.url });
            alert('Imagen subida exitosamente');
        } else {
            alert('Error al subir imagen');
        }
    };
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        
        // Enviar datos con URL de imagen
        const result = await productoService.create(formData);
        
        if (result.success) {
            alert('Producto creado');
        }
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <div>
                <label>Nombre:</label>
                <input
                    type="text"
                    value={formData.nombre}
                    onChange={(e) => setFormData({ ...formData, nombre: e.target.value })}
                    required
                />
            </div>
            
            <div>
                <label>Imagen:</label>
                <input
                    type="file"
                    accept="image/*"
                    onChange={handleFileChange}
                    disabled={uploading}
                />
                {uploading && <span>Subiendo...</span>}
                {preview && (
                    <img src={preview} alt="Preview" style={{ maxWidth: 200, marginTop: 10 }} />
                )}
            </div>
            
            <button type="submit" disabled={uploading}>
                Crear Producto
            </button>
        </form>
    );
}
```

---

## Drag & Drop

```jsx
// src/components/ImageUpload.jsx
import { useState } from 'react';
import { uploadService } from '../services/uploadService';

export default function ImageUpload({ onUpload }) {
    const [dragging, setDragging] = useState(false);
    const [uploading, setUploading] = useState(false);
    const [preview, setPreview] = useState(null);
    
    const handleDragEnter = (e) => {
        e.preventDefault();
        setDragging(true);
    };
    
    const handleDragLeave = (e) => {
        e.preventDefault();
        setDragging(false);
    };
    
    const handleDrop = async (e) => {
        e.preventDefault();
        setDragging(false);
        
        const file = e.dataTransfer.files[0];
        if (file && file.type.startsWith('image/')) {
            await uploadImage(file);
        }
    };
    
    const handleFileChange = async (e) => {
        const file = e.target.files[0];
        if (file) {
            await uploadImage(file);
        }
    };
    
    const uploadImage = async (file) => {
        // Preview
        const reader = new FileReader();
        reader.onloadend = () => {
            setPreview(reader.result);
        };
        reader.readAsDataURL(file);
        
        // Upload
        setUploading(true);
        const result = await uploadService.uploadImage(file);
        setUploading(false);
        
        if (result.success) {
            onUpload(result.url);
        } else {
            alert('Error al subir imagen');
        }
    };
    
    return (
        <div>
            <div
                onDragEnter={handleDragEnter}
                onDragOver={(e) => e.preventDefault()}
                onDragLeave={handleDragLeave}
                onDrop={handleDrop}
                style={{
                    border: `2px dashed ${dragging ? '#007bff' : '#ccc'}`,
                    padding: '2rem',
                    textAlign: 'center',
                    borderRadius: '8px',
                    backgroundColor: dragging ? '#f0f8ff' : 'transparent',
                    cursor: 'pointer',
                }}
            >
                <input
                    type="file"
                    accept="image/*"
                    onChange={handleFileChange}
                    style={{ display: 'none' }}
                    id="file-input"
                    disabled={uploading}
                />
                <label htmlFor="file-input" style={{ cursor: 'pointer' }}>
                    {uploading ? (
                        <p>Subiendo...</p>
                    ) : preview ? (
                        <img src={preview} alt="Preview" style={{ maxWidth: 300 }} />
                    ) : (
                        <div>
                            <p>📁 Arrastra una imagen aquí</p>
                            <p>o haz clic para seleccionar</p>
                        </div>
                    )}
                </label>
            </div>
        </div>
    );
}
```

**Uso:**
```jsx
<ImageUpload onUpload={(url) => setFormData({ ...formData, imagen: url })} />
```

---

## Múltiples imágenes

### Backend (Django)

```python
# productos/models.py
class Producto(models.Model):
    nombre = models.CharField(max_length=200)
    # ...

class ProductoImagen(models.Model):
    producto = models.ForeignKey(Producto, on_delete=models.CASCADE, related_name='imagenes')
    imagen = models.ImageField(upload_to='productos/')
    orden = models.IntegerField(default=0)
    
    class Meta:
        ordering = ['orden']
```

```python
# productos/serializers.py
class ProductoImagenSerializer(serializers.ModelSerializer):
    class Meta:
        model = ProductoImagen
        fields = ['id', 'imagen', 'orden']

class ProductoSerializer(serializers.ModelSerializer):
    imagenes = ProductoImagenSerializer(many=True, read_only=True)
    
    class Meta:
        model = Producto
        fields = ['id', 'nombre', 'imagenes', ...]
```

### Frontend (React)

```jsx
// src/components/MultipleImageUpload.jsx
import { useState } from 'react';
import { uploadService } from '../services/uploadService';

export default function MultipleImageUpload({ onUpload }) {
    const [images, setImages] = useState([]);
    const [uploading, setUploading] = useState(false);
    
    const handleFilesChange = async (e) => {
        const files = Array.from(e.target.files);
        
        setUploading(true);
        
        const uploadPromises = files.map(file => uploadService.uploadImage(file));
        const results = await Promise.all(uploadPromises);
        
        const urls = results
            .filter(r => r.success)
            .map(r => r.url);
        
        setImages([...images, ...urls]);
        onUpload([...images, ...urls]);
        
        setUploading(false);
    };
    
    const removeImage = (index) => {
        const newImages = images.filter((_, i) => i !== index);
        setImages(newImages);
        onUpload(newImages);
    };
    
    return (
        <div>
            <input
                type="file"
                accept="image/*"
                multiple
                onChange={handleFilesChange}
                disabled={uploading}
            />
            
            {uploading && <p>Subiendo...</p>}
            
            <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '1rem', marginTop: '1rem' }}>
                {images.map((url, index) => (
                    <div key={index} style={{ position: 'relative' }}>
                        <img src={url} alt={`Imagen ${index + 1}`} style={{ width: '100%', height: 150, objectFit: 'cover' }} />
                        <button
                            onClick={() => removeImage(index)}
                            style={{
                                position: 'absolute',
                                top: 5,
                                right: 5,
                                background: 'red',
                                color: 'white',
                                border: 'none',
                                borderRadius: '50%',
                                width: 25,
                                height: 25,
                                cursor: 'pointer',
                            }}
                        >
                            ✕
                        </button>
                    </div>
                ))}
            </div>
        </div>
    );
}
```

---

## Progreso de upload

```javascript
// src/services/uploadService.js
export const uploadService = {
    uploadWithProgress: (file, onProgress) => {
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            
            const cloudName = import.meta.env.VITE_CLOUDINARY_CLOUD_NAME;
            const uploadPreset = import.meta.env.VITE_CLOUDINARY_UPLOAD_PRESET;
            
            xhr.open('POST', `https://api.cloudinary.com/v1_1/${cloudName}/image/upload`);
            
            // Progress
            xhr.upload.addEventListener('progress', (e) => {
                if (e.lengthComputable) {
                    const percentComplete = Math.round((e.loaded / e.total) * 100);
                    onProgress(percentComplete);
                }
            });
            
            // Success
            xhr.addEventListener('load', () => {
                if (xhr.status === 200) {
                    const data = JSON.parse(xhr.responseText);
                    resolve({
                        success: true,
                        url: data.secure_url,
                        publicId: data.public_id,
                    });
                } else {
                    reject({ success: false, error: 'Error al subir' });
                }
            });
            
            // Error
            xhr.addEventListener('error', () => {
                reject({ success: false, error: 'Error de red' });
            });
            
            // Send
            const formData = new FormData();
            formData.append('file', file);
            formData.append('upload_preset', uploadPreset);
            xhr.send(formData);
        });
    },
};
```

**Uso:**
```jsx
const [progress, setProgress] = useState(0);

const handleUpload = async (file) => {
    try {
        const result = await uploadService.uploadWithProgress(file, setProgress);
        console.log('Subido:', result.url);
    } catch (error) {
        console.error('Error:', error);
    }
};

return (
    <div>
        <input type="file" onChange={(e) => handleUpload(e.target.files[0])} />
        {progress > 0 && (
            <div>
                <progress value={progress} max="100" />
                <span>{progress}%</span>
            </div>
        )}
    </div>
);
```

---

## Validaciones

### Backend

```python
# productos/serializers.py
from rest_framework import serializers

class ProductoSerializer(serializers.ModelSerializer):
    def validate_imagen(self, value):
        # Validar tamaño (máximo 5MB)
        if value.size > 5 * 1024 * 1024:
            raise serializers.ValidationError("La imagen no puede superar 5MB")
        
        # Validar tipo
        if not value.content_type.startswith('image/'):
            raise serializers.ValidationError("Solo se permiten imágenes")
        
        return value
```

### Frontend

```javascript
const validateImage = (file) => {
    // Tamaño máximo: 5MB
    if (file.size > 5 * 1024 * 1024) {
        alert('La imagen no puede superar 5MB');
        return false;
    }
    
    // Tipo
    if (!file.type.startsWith('image/')) {
        alert('Solo se permiten imágenes');
        return false;
    }
    
    return true;
};

const handleFileChange = async (e) => {
    const file = e.target.files[0];
    if (!file || !validateImage(file)) return;
    
    // Upload...
};
```

---

## Resumen

**Estrategias:**
- Local (desarrollo) → `MEDIA_ROOT`
- Cloud (producción) → Cloudinary, S3

**Backend (Django):**
- `ImageField` en modelo
- `cloudinary-django-storage` para cloud

**Frontend (React):**
- FormData para uploads
- Upload directo a Cloudinary (sin pasar por backend)
- Drag & Drop
- Múltiples imágenes
- Barra de progreso

**Próximo módulo:** Deployment conjunto (Vercel + Railway)

---

## Recursos

- **[Cloudinary Docs](https://cloudinary.com/documentation)** - Documentación oficial
- **[Django File Uploads](https://docs.djangoproject.com/en/5.0/topics/http/file-uploads/)** - Guía oficial
- **[FormData API](https://developer.mozilla.org/en-US/docs/Web/API/FormData)** - MDN

¡Upload de archivos implementado! 📤
