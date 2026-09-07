# Amazon Elastic Container Registry (ECR)
Amazon ECR es un servicio de registro de contenedores Docker administrado por AWS. Permite almacenar, administrar y desplegar imágenes de contenedores de forma segura y escalable.

## 📘 ¿Qué es ECR y por qué lo usamos?
En el desarrollo tradicional, usas Docker Hub para subir tus imágenes de contenedor. En entornos empresariales dentro de AWS, usar Docker Hub introduce problemas de latencia, costos por transferencia de datos y riesgos de seguridad. ECR resuelve esto al integrarse de forma nativa con el ecosistema de AWS (IAM, ECS, EKS, Lambda).

### Componentes Clave
- **Registro (Registry)**: Tu cuenta de AWS tiene un registro por defecto. Aloja múltiples repositorios.
- **Repositorio (Repository)**: El lugar específico donde subes las versiones (tags) de una imagen de contenedor (ej. mi-api-backend).
- **Políticas de Ciclo de Vida (Lifecycle Policies)**: Reglas automáticas para borrar imágenes viejas y no pagar almacenamiento de más.
- **Escaneo de Vulnerabilidades**: ECR analiza tus imágenes en busca de fallos de seguridad en el código OS o dependencias.


## 🛠️ Guía Práctica: Crear y Subir tu Primera Imagen
Aprenderás a crear un repositorio en ECR, autenticarte y subir una imagen Docker desde tu terminal.
### Prerrequisitos
1. Tener instalado AWS CLI y Docker.
2. Configurar tus credenciales de AWS con aws configure (tu usuario requiere permisos de ECR).

### Paso 1: Crear el Repositorio desde la Terminal
Ejecuta el siguiente comando para crear un repositorio privado llamado junior-express-app:
```bash
aws ecr create-repository \
    --repository-name express-app \
    --image-scanning-configuration scanOnPush=true \
    --region us-east-1
```

Guarda el valor de `repositoryUri` que aparece en la respuesta de la terminal (ej. amazonaws.com).

### Paso 2: Autenticar Docker con AWS ECR
ECR requiere un token de acceso temporal que expira cada 12 horas. Obtén el token y pásalo a Docker con este comando:
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin amazonaws.com
```

**IMPORTANTE**: Reemplaza `123456789012` con tu ID de cuenta de AWS real.

### Paso 3: Construir y Etiquetar (Tag) la Imagen Local
Asumiendo que tienes un Dockerfile en tu directorio actual, construye la imagen:

```bash
docker build -t express-app .
```

Ahora, ponle una etiqueta que apunte a tu repositorio de ECR. Para ello debes indicar donde aparece `amazonaws.com` a qué servidor de AWS, a qué cuenta y a qué repositorio debe enviar los archivos. La estructura del tag obligatorio de AWS ECR es:
`[ID_DE_CUENTA].dkr.ecr.[REGION]://[NOMBRE_REPOSITORIO]:[TAG_VERSION]`

```bash
docker tag express-app:latest amazonaws.com
```

### Paso 4: Subir la Imagen (Push)
Envía la imagen empaquetada a la nube de AWS:

```bash
docker push amazonaws.com
```

**OJO:** Ten en cuenta lo mismo del paso anterior, reemplazar `amazonaws.com` por la estructura del tag que se espera (`[ID_DE_CUENTA].dkr.ecr.[REGION]://[NOMBRE_REPOSITORIO]:[TAG_VERSION]`)

¡Listo! Si vas a la consola web de AWS ECR, verás tu imagen v1.0.0 almacenada de forma segura.

## 💡 Pro Tips
- **⚠️ Evita usar el tag :latest en producción**: El tag :latest es un peligro silencioso. Si subes una imagen rota con ese tag, tus servicios (ECS/EKS) la descargarán automáticamente al reiniciarse, rompiendo producción. Usa siempre versionamiento semántico (v1.0.0) o el hash de Git commit (git-a1b2c3d).
- **💰 Implementa Lifecycle Policies desde el día uno**: Cada capa de Docker pesa megabytes o gigabytes. Si tu CI/CD sube imágenes en cada commit, tu factura de AWS crecerá sin control. Configura una política de ciclo de vida en ECR para mantener únicamente las últimas 10 imágenes y eliminar las demás.
- **🔒 Activa la Mutabilidad de Tags como "Immutable"**: Configura tu repositorio como IMMUTABLE. Esto evita que alguien suba por error una imagen nueva con un tag existente (ej. sobreescribir v1.0.0), garantizando que el código desplegado sea exactamente el que se probó.
- **🚀 Habilita "Scan on Push"**: No subas contenedores a ciegas. Al activar el escaneo automático, AWS utilizará la base de datos de Clair para avisarte si tu imagen base tiene vulnerabilidades críticas de seguridad antes de que llegue a tus servidores.