# 🏠 UniHogar — Plataforma de Arriendos Estudiantiles

> 🎓 **Proyecto Académico** | Universidad Estatal de Bolívar  
> • **Carrera:** Ingeniería de Software  
> • **Materia:** Desarrollo de Software Móvil  
> • **Ciclo:** 6to Ciclo "A"  
> • **Equipo:** Sebastian Velastegui & Neicer Jimenez  

---

## 📌 Visión General

**UniHogar** es una solución integral diseñada para modernizar y centralizar la oferta de arriendos (habitaciones, departamentos y casas) para los estudiantes de la **Universidad Estatal de Bolívar (UEB)** en Guaranda, Ecuador. 

El ecosistema conecta a estudiantes universitarios con arrendadores verificados a través de una experiencia móvil nativa, complementada con un panel web administrativo para la moderación y un backend con persistencia en PostgreSQL sobre contenedores Docker.

---

## 🏗️ Arquitectura del Ecosistema

El proyecto está dividido en tres componentes especializados e independientes:

```
                          ┌─────────────────────────────┐
                          │   UniHogar — Ecosistema     │
                          └──────────────┬──────────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        ▼                                ▼                                ▼
┌────────────────────────┐    ┌────────────────────────┐    ┌────────────────────────┐
│   📱 App Móvil Android │    │ 💻 Panel Web Admin     │    │  🗄️ Base de Datos      │
│  (Catálogo Estudiantes)│    │  (Gestión y Auditoría) │    │  (PostgreSQL + Docker) │
│                        │    │                        │    │                        │
│ • Búsqueda & filtros   │    │ • Validación de perfiles│    │ • Esquema relacional   │
│ • Georreferenciación   │    │ • Moderación de avisos │    │ • Adminer Web UI       │
│ • Comparador de arriendo│   │ • Estadísticas globales│    │ • Scripts de seed      │
└────────────────────────┘    └────────────────────────┘    └────────────────────────┘
```

---

## 📦 Componentes del Sistema

| Módulo | Repositorio | Visibilidad | Tecnologías | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **App Móvil** | [**`unihogar-guaranda`**](https://github.com/CeoF10/unihogar-guaranda) | **Público** | Android Nativo, Kotlin/Java | Aplicación orientada a estudiantes para consultar catálogo de viviendas, ubicación y contacto directo con arrendadores. |
| **Panel Web** | [`unihogar-admin-web`](https://github.com/CeoF10/unihogar-admin-web) | **Privado** | HTML5, CSS3, JavaScript | Dashboard para moderadores y administradores: revisión de publicaciones, gestión de estados y auditoría. |
| **Persistencia** | [`unihogar-database`](https://github.com/CeoF10/unihogar-database) | **Privado** | PostgreSQL 16, Docker Compose, Adminer | Definición del esquema relacional, migraciones y servicios de base de datos en contenedores. |

---

## 🚀 Inicio Rápido Local

Para levantar el entorno completo en local:

### 1. Iniciar Base de Datos y Servicios
```bash
cd unihogar-database
docker compose up -d
```
* Acceso a consola Adminer: `http://localhost:8085`

### 2. Abrir el Panel Web Administrativo
```bash
cd ../unihogar-admin-web
# Abrir index.html en tu navegador o mediante un servidor local (Live Server / Python)
python3 -m http.server 8080
```

### 3. Compilar la App Móvil (Android Studio)
```bash
cd ../unihogar-guaranda
# Abrir el directorio en Android Studio y sincronizar Gradle
```

---

## 🔒 Seguridad y Privacidad

- **Verificación institucional:** Los perfiles de estudiantes se asocian al dominio institucional `@ueb.edu.ec`.
- **Protección de datos:** Los componentes de persistencia y administración se gestionan de forma privada para salvaguardar credenciales y registros de arrendadores y estudiantes.
