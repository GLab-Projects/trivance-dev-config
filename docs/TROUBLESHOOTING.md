# 🆘 Guía de Solución de Problemas

## 🔧 Problemas de Setup

### npm install se cuelga o falla
```bash
# Limpiar cache de npm
npm cache clean --force

# Usar registro oficial
npm config set registry https://registry.npmjs.org/

# Reinstalar desde cero
rm -rf node_modules package-lock.json
npm install
```

### Git clone falla
```bash
# Verificar acceso GitHub
ssh -T git@github.com

# Usar HTTPS si SSH falla
git config --global url."https://".insteadOf git://
```

## 🚀 Problemas de Servicios

### Servicio no inicia con PM2
```bash
# Ver logs del servicio
pm2 logs [nombre-servicio]

# Verificar puerto ocupado
lsof -i :[puerto]
kill -9 [PID]

# Reiniciar servicio
pm2 restart [nombre-servicio]
```

### Todos los servicios fallan
```bash
# Verificar Node.js version
node --version  # Debe ser 18+

# Validar configuración
./trivance-dev-config/scripts/envs.sh validate

# Verificar contenedores Docker
docker ps
docker logs trivance_postgres
docker logs trivance_mongodb
```

## 🗄️ Problemas de Base de Datos

### PostgreSQL no conecta
```bash
# Verificar que esté corriendo
pg_isready

# Verificar string de conexión en .env
DATABASE_URL=postgresql://user:pass@localhost:5432/db

# Crear base de datos si no existe
psql postgres
CREATE DATABASE trivance_development;
```

### MongoDB no conecta
```bash
# Verificar MongoDB
mongosh --eval "db.adminCommand('ping')"

# Verificar string de conexión
DB_MONGO=mongodb://localhost:27017/trivance_auth
```

## 🐳 Problemas de Docker

### Docker no está instalado o no corre
```bash
# Verificar instalación
docker --version

# Si no está instalado, instalar Docker Desktop desde:
# https://www.docker.com/products/docker-desktop/

# Verificar que Docker esté corriendo
docker ps
```

### Contenedores no inician
```bash
# Ver logs específicos
docker logs trivance_management
docker logs trivance_auth

# Recrear contenedores
cd trivance-dev-config/docker
docker compose down
docker compose up -d

# Verificar archivos .env Docker
ls -la trivance-dev-config/docker/.env.docker-*
```

### Error de conexión a base de datos
```bash
# Verificar que las URLs usen hostnames Docker
grep "DATABASE_URL\|DB_MONGO" trivance-dev-config/docker/.env.docker-*

# Debe mostrar:
# DATABASE_URL=postgresql://...@postgres:5432/...
# DB_MONGO=mongodb://mongodb:27017/...

# Si no, regenerar:
./trivance-dev-config/scripts/envs.sh switch local
```

### Error "fetch failed" en UnifiedLogger
Si ves errores como:
```
[Nest] 29  - 07/19/2025, 3:03:59 PM    WARN [UnifiedLogger] Error sending log to unified logger: fetch failed
```

**Causa**: Auth Service no puede conectarse a Management API.

**Solución**: El archivo `docker-compose.dev.yml` incluye un network alias para resolver este problema:
```yaml
ms_level_up_management:
  networks:
    trivance_dev:
      aliases:
        - trivance_management  # Alias necesario para Auth Service
```

**Verificar**:
```bash
# Comprobar que el alias está funcionando
docker exec trivance_auth_dev ping trivance_management

# Ver logs del Management API
docker logs trivance_mgmt_dev | grep "health"
```

## ⚙️ Problemas de PM2 (Solo Frontend)

### PM2 no encontrado
```bash
# Instalar PM2 globalmente
npm install -g pm2

# Verificar PATH
export PATH=$PATH:$(npm config get prefix)/bin
```

### Logs vacíos
```bash
# Reiniciar PM2 logs
pm2 flush
pm2 restart all

# Verificar permisos de logs
ls -la logs/pm2/
```

## 🌐 Problemas de Frontend

### Frontend no accesible (puerto 5173)
```bash
# Verificar puerto en logs
pm2 logs backoffice | grep "Local:"

# Limpiar cache de Vite
cd level_up_backoffice
rm -rf node_modules/.vite
pm2 restart backoffice
```

### GraphQL Playground no carga
```bash
# Verificar Management API
pm2 status management-api

# Verificar NODE_ENV
echo $NODE_ENV  # Debe ser 'development'
```

## 📱 Problemas de Mobile

### Metro bundler falla
```bash
cd trivance-mobile

# Limpiar cache Metro
npx react-native start --reset-cache

# Limpiar watchman
watchman watch-del-all
```

## 🔄 Problemas de Environment

### Environment no cambia
```bash
# Reiniciar servicios después de cambio
pm2 restart all

# Verificar archivos .env copiados
ls -la */. env

# Forzar recarga
pm2 delete all
./start.sh start
```

## ⚡ Soluciones Rápidas

### Script de Fixes Automáticos
```bash
# Ejecutar fixes post-setup
./trivance-dev-config/scripts/utils/post-setup-fixes.sh
```

### Reset de Emergencia
```bash
# CUIDADO: Esto borra todo
pm2 kill
./trivance-dev-config/scripts/utils/clean-workspace.sh
./trivance-dev-config/setup.sh
```

### Verificación Completa
```bash
# Validar todo el setup
./trivance-dev-config/scripts/utils/validate-setup.sh

# Verificar compilación
./trivance-dev-config/scripts/utils/validate-setup.sh
```

## 🚨 Errores Comunes Específicos

### "Cannot find module"
```bash
cd [repositorio-afectado]
rm -rf node_modules package-lock.json
npm install
pm2 restart [servicio]
```

### "EADDRINUSE: address already in use"
```bash
# El setup maneja esto automáticamente
pm2 stop all
pm2 start ecosystem.config.js
```

### "Database connection error"
```bash
# Verificar servicios de BD
# PostgreSQL: puerto 5432
# MongoDB: puerto 27017

# macOS
brew services list | grep -E "(postgresql|mongodb)"

# Linux
systemctl status postgresql
systemctl status mongod
```

### Timeout en compilación
```bash
# Limpiar cache y recompilar
rm -rf node_modules package-lock.json
npm install
npm run build
```

## 📞 Obtener Ayuda

### Información del Sistema
```bash
# Report completo PM2
pm2 report

# Logs detallados
pm2 logs --lines 100

# Debug mode
DEBUG=* pm2 restart [servicio]
```

### Soporte del Equipo
- **Slack**: #dev-support
- **Documentación**: `docs/` folder
- **GitHub Issues**: Para bugs del sistema
- **Team Lead**: Para problemas bloqueantes

---

**💡 Tip**: La mayoría de problemas se resuelven con `pm2 restart all` y verificar que las bases de datos estén corriendo.