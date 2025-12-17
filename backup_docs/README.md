# Sistema de Gestión de Fotos y Pedidos - Torneo Tres Fronteras

## 📋 Descripción
Sistema integral para la gestión de fotos y pedidos del Torneo Tres Fronteras, desarrollado con Laravel. Permite a los padres visualizar y comprar fotos de los partidos de sus hijos de manera sencilla, mientras facilita la gestión de pedidos y entregas para los organizadores.

## 🎯 Características Principales

### Para Padres/Clientes
- 📱 Interfaz intuitiva para visualizar fotos por categoría, equipo y partido
- 🔍 Búsqueda avanzada de fotos por múltiples criterios
- 🛒 Carrito de compras con seguimiento en tiempo real
- 📲 Múltiples opciones de entrega: físico, digital o combinado
- 🎟️ Sistema de tickets para no perder el progreso

### Para Operadores
- 📊 Panel de control para gestión de pedidos
- 💳 Seguimiento de pagos y estados
- 🖨️ Control de impresión y preparación de pedidos
- 📊 Reportes y estadísticas

## 🛠️ Tecnologías
- **Backend:** PHP 8.x, Laravel 10.x
- **Frontend:** Blade, Tailwind CSS, Livewire
- **Base de Datos:** MySQL/PostgreSQL
- **Almacenamiento:** Sistema de archivos local/S3
- **Colas:** Redis/Beanstalkd para tareas asíncronas

## 🚀 Requisitos del Sistema
- PHP 8.1 o superior
- Composer 2.x
- Node.js 16+ y NPM 8+
- MySQL 8.0+ o PostgreSQL 13+
- Servidor web (Apache/Nginx) con soporte para URL rewrite
- Extensión PHP Fileinfo habilitada
- Extensión PHP GD/Imagick para procesamiento de imágenes

## 📦 Instalación

1. Clonar el repositorio:
```bash
git clone [URL_DEL_REPOSITORIO]
cd tres_fronteras_laravel
```

2. Instalar dependencias de PHP y Node.js:
```bash
composer install
npm install
npm run build
```

3. Configurar el entorno:
```bash
cp .env.example .env
php artisan key:generate
```

4. Configurar la base de datos en el archivo `.env`

5. Ejecutar migraciones y seeders:
```bash
php artisan migrate --seed
```

6. Configurar el almacenamiento:
```bash
php artisan storage:link
```

7. Iniciar el servidor de desarrollo:
```bash
php artisan serve
```

## 🔒 Variables de Entorno

Asegúrate de configurar las siguientes variables en tu archivo `.env`:

```
APP_ENV=local
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=tres_fronteras
DB_USERNAME=root
DB_PASSWORD=

QUEUE_CONNECTION=sync  # Cambiar a database/redis en producción

FILESYSTEM_DISK=public  # Cambiar a s3 en producción

# Configuración de WhatsApp (si aplica)
WHATSAPP_API_KEY=
WHATSAPP_PHONE_NUMBER=
```

## 🧑‍💻 Uso

### Roles de Usuario
- **Administrador**: Acceso completo al sistema
- **Operador**: Gestión de pedidos y entregas
- **Cliente**: Visualización y compra de fotos

### Comandos Artisan Útiles
```bash
# Procesar colas de trabajo
php artisan queue:work

# Limpiar caché
php artisan optimize:clear

# Generar documentación
php artisan docs:generate
```

## 🤝 Contribución

1. Hacer fork del proyecto
2. Crear una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Hacer commit de tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Hacer push a la rama (`git push origin feature/AmazingFeature`)
5. Abrir un Pull Request

## 📄 Licencia
Este proyecto está bajo la Licencia MIT. Ver el archivo `LICENSE` para más detalles.

## 📞 Contacto
Para soporte o consultas, contactar al equipo de desarrollo.

---

<div align="center">
  <sub>Creado con ❤️ para el Torneo Tres Fronteras</sub>
</div>
