# Patitas en Espera — Product Requirements Document

## Visión general

Plataforma web para gestionar el tránsito temporal de mascotas rescatadas en Argentina. Conecta rescatistas/organizaciones con personas dispuestas a ofrecer un hogar temporal, con seguimiento integral de cada mascota durante todo su proceso.

**MVP geográfico:** Santa Fe Capital y Rosario  
**Expansión futura:** Top 5 ciudades → todo Argentina (sin cambios técnicos, solo operativos)

---

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Frontend | Next.js 14 (App Router) + TypeScript |
| Styling | Tailwind CSS + shadcn/ui |
| Backend | Next.js API Routes + Server Actions |
| Base de datos | Supabase (PostgreSQL) |
| Auth | Supabase Auth (OAuth Google/Facebook) |
| Storage | Supabase Storage (fotos, videos, documentos) |
| Realtime | Supabase Realtime (chat, notificaciones) |
| MCP | @supabase/mcp-server-supabase |
| Deploy | Vercel |

---

## Roles y actores

### 1. Transitante
Persona que ofrece hogar temporal a una mascota.
- Se registra vía OAuth
- **Obligatorio antes de postularse:** completar formulario de hogar (ver sección)
- Puede postularse a mascotas disponibles
- Sube updates periódicos durante el tránsito
- Puede solicitar adopción definitiva

### 2. Rescatista Independiente
Persona individual que rescata mascotas.
- Carga mascotas al sistema
- Gestiona postulaciones para sus mascotas
- Aprueba/rechaza transitantes
- Sigue el estado de cada animal

### 3. Organización / ONG
Entidad con múltiples rescatistas y mascotas.
- Panel de organización propio
- Múltiples miembros con distintos permisos
- Gestión centralizada de mascotas y transitantes
- Verificada por superadmin antes de poder operar

### 4. Veterinario Verificado
Profesional habilitado para validar información médica.
- Carga y firma historia clínica de mascotas
- Registra vacunas, cirugías, diagnósticos
- Su firma digital da credibilidad al historial médico

### 5. Superadmin
Administrador de la plataforma.
- Gestión de usuarios (ban, warnings)
- Verificación y aprobación de Organizaciones
- Verificación de Veterinarios
- Panel de reportes de abuso
- Métricas básicas de la plataforma

---

## Flujos principales

### Flujo de registro de usuario

```
1. Usuario accede a la app
2. Login vía OAuth (Google o Facebook)
3. Selecciona su rol: Transitante | Rescatista | Organización
4. Completa perfil básico (nombre, ciudad, teléfono)
5. Si es Transitante → redirige al Formulario de Hogar (obligatorio)
6. Si es Rescatista/Org → puede empezar a cargar mascotas
```

### Formulario de Hogar (Transitante) — campos completos

**Datos del hogar:**
- Tipo de vivienda: casa / depto / PH
- Metros cuadrados aproximados
- ¿Tiene patio o espacio exterior? → tamaño estimado
- Piso (si es depto) + ascensor
- ¿Reja o cercado en el patio?
- ¿Alquila o es propietario? (si alquila, ¿el dueño permite mascotas?)
- Ciudad y barrio
- Fotos del hogar (obligatorias: al menos 3, incluyendo espacio donde estaría la mascota)

**Convivencia:**
- ¿Tiene hijos? → edades de cada uno
- ¿Tiene otras mascotas? → tipo, raza, tamaño, sexo, castrado/vacunado de cada una
- ¿Cuántas horas al día la mascota estaría sola?
- ¿Trabaja desde casa o fuera?

**Capacidad económica:**
- ¿Puede cubrir gastos veterinarios de urgencia? (sí / parcialmente / no)
- ¿Cuenta con transporte para llevar al vet?
- ¿Recibiría subsidio del rescatista/org? (si aplica)

**Experiencia:**
- ¿Tuvo mascotas antes? → descripción
- ¿Experiencia con animales con trauma o maltrato?
- Tamaño de mascota que puede alojar: pequeño / mediano / grande / gigante
- Especies que puede alojar: perros / gatos / ambos / otros
- ¿Puede transitar mascotas que requieren medicación?

**Estado del formulario:**
- `pendiente_revision` → `aprobado` / `rechazado` (por el rescatista/org al que se postula)

---

### Flujo de carga de mascota (Rescatista/Org)

```
1. Rescatista crea perfil de la mascota
2. Carga datos básicos + compatibilidades + historia de rescate
3. Sube fotos y/o videos
4. Opcionalmente invita a vet verificado para cargar historia clínica
5. Publica la mascota como "disponible para tránsito"
6. La mascota aparece en el feed público de su ciudad
```

### Perfil de mascota — campos completos

**Datos básicos:**
- Nombre (o apodo provisional)
- Especie: perro / gato / otro
- Raza (o "mestizo")
- Edad estimada
- Sexo
- Tamaño: pequeño (<8kg) / mediano (8-20kg) / grande (20-40kg) / gigante (>40kg)
- Castrado/esterilizado: sí / no / en proceso
- Color y descripción física

**Compatibilidades:**
- Compatible con niños: sí / depende edad / no
- Compatible con perros: sí / depende / no
- Compatible con gatos: sí / depende / no
- Nivel de energía: bajo / moderado / alto
- Necesita espacio exterior
- Necesita ser única mascota

**Historia clínica** (cargada por rescatista o vet verificado):
- Vacunas al día: sí / no / parcial
- Desparasitación: sí / no
- Enfermedades activas o crónicas
- Cirugías previas
- Medicación actual (tipo y frecuencia)
- Alergias conocidas

**Historia de rescate:**
- Situación de rescate: abandono / maltrato / accidente / nacido en calle / entrega voluntaria
- Descripción de la historia
- Estado emocional: equilibrado / ansioso / miedoso / en proceso de socialización
- Convivió con humanos antes: sí / no / desconocido

**Estado de la mascota:**
- `disponible` → `en_postulacion` → `en_transito` → `adoptado` / `devuelto` / `fallecido`

---

### Flujo de postulación

```
1. Transitante navega el feed, filtra por ciudad / especie / tamaño / compatibilidad
2. Ve el perfil de una mascota → lee toda la info
3. Hace click en "Postularme"
   - Si no tiene formulario de hogar completo → bloqueado, debe completarlo primero
4. El rescatista/org recibe notificación de nueva postulación
5. Rescatista ve el perfil completo del transitante (hogar, fotos, compatibilidades, formulario)
6. Puede: Aprobar / Rechazar / Iniciar chat para consultar
7. Si hay múltiples postulantes → el rescatista elige uno
8. Al aprobar: ambas partes reciben info de contacto real (teléfono/email)
9. Se acuerda entrega → rescatista registra fecha de inicio de tránsito
10. Estado de la mascota pasa a `en_transito`
```

---

### Flujo de tránsito activo

```
1. Sistema genera check-ins automáticos según frecuencia (configurable por rescatista: semanal / quincenal)
2. Transitante recibe notificación → sube foto + nota de estado
3. Si no responde en 48h → alerta al rescatista
4. Transitante puede registrar visitas al vet (fecha, motivo, costo, diagnóstico)
5. Rescatista puede ver todo el historial en tiempo real
6. Timeline público de la mascota (si rescatista lo habilita) → compartible en redes
```

---

### Flujo de cierre de tránsito

Al acercarse la fecha de fin, el sistema notifica a ambas partes. El rescatista elige:

| Salida | Descripción |
|--------|-------------|
| **Adopción definitiva** | Transitante solicita adoptar. Rescatista aprueba. Mascota pasa a `adoptado`. |
| **Nuevo tránsito** | Mascota vuelve al pool disponible. El historial completo queda visible. |
| **Devolución a org** | Mascota vuelve físicamente a la org/rescatista. Queda en `en_refugio`. |
| **Extensión** | Transitante y rescatista acuerdan extender. Nueva fecha registrada. |

---

## Sistema de comunicación

**Etapa de postulación:** Chat interno en la app (ambas partes pueden intercambiar preguntas)  
**Postulación aprobada:** Se comparte contacto real (teléfono/email) dentro de la plataforma  
**Durante el tránsito:** Las actualizaciones fluyen por la app (fotos, notas, check-ins). El canal informal es externo.

---

## Timeline público de la mascota

Cada mascota tiene un perfil público con:
- Nombre, fotos, historia de rescate (versión editada por el rescatista)
- Estado actual: buscando tránsito / en tránsito / adoptado
- Updates del transitante (solo las que el rescatista habilita como públicas)
- Compartible como link en redes sociales

---

## Donaciones (Fase 2)

**No incluido en MVP. Preparar la arquitectura para integrar luego:**
- Donaciones específicas a una mascota para cubrir gastos veterinarios
- Integración con MercadoPago (Argentina)
- El dinero recaudado va al rescatista/org de la mascota
- Historial transparente de gastos y uso del dinero

---

## Panel Superadmin (MVP básico)

- Lista de usuarios con filtros (rol, ciudad, estado)
- Ban/unban de usuarios
- Lista de organizaciones pendientes de verificación → aprobar/rechazar
- Lista de veterinarios pendientes de verificación
- Reportes de abuso recibidos → acción
- Métricas básicas: total mascotas, en tránsito, adoptadas, usuarios activos

---

## Cobertura geográfica

**MVP:** Santa Fe Capital y Rosario  
**Expansión:** Agregar ciudades a la tabla `cities` — sin cambios en la lógica de la app  
**Nota técnica:** El filtro geográfico opera sobre el campo `city_id` de usuarios y mascotas. Agregar CABA/AMBA es solo insertar registros en la tabla de ciudades y abrir el registro en esas localidades.

---

## Arquitectura de datos — esquema preliminar (Supabase/PostgreSQL)

```sql
-- Geografía
cities (id, name, province, country)

-- Usuarios
profiles (id, user_id, role, full_name, phone, city_id, avatar_url, created_at)

-- Organizaciones
organizations (id, name, description, city_id, verified, admin_user_id)
org_members (org_id, user_id, role)

-- Formulario de hogar del transitante
foster_profiles (
  id, user_id,
  home_type, sqm, has_yard, yard_size, floor, has_elevator, has_fence,
  rents, landlord_allows_pets,
  city_id, neighborhood, home_photos[],
  has_children, children_ages[],
  other_pets jsonb,
  hours_alone, works_from_home,
  can_cover_vet_costs, has_transport,
  had_pets_before, trauma_experience,
  max_pet_size, pet_species[],
  can_medicate,
  status -- pending | approved | rejected
)

-- Mascotas
pets (
  id, name, species, breed, age_estimate, sex, size, color, neutered,
  compatible_children, compatible_dogs, compatible_cats,
  energy_level, needs_outdoor, needs_alone,
  rescue_situation, emotional_state, lived_with_humans,
  rescue_description,
  vaccinated, dewormed, current_medication, allergies, diseases,
  status, -- available | in_postulation | in_transit | adopted | returned | deceased
  rescuer_id, org_id,
  city_id,
  public_profile_enabled,
  created_at
)
pet_photos (id, pet_id, url, is_main)

-- Historia clínica
medical_records (id, pet_id, vet_id, date, type, description, cost)

-- Postulaciones
applications (
  id, pet_id, foster_user_id,
  status, -- pending | approved | rejected | withdrawn
  message,
  rescuer_notes,
  created_at
)

-- Tránsitos activos
transits (
  id, pet_id, foster_user_id, rescuer_id,
  start_date, end_date, extended_end_date,
  checkin_frequency, -- weekly | biweekly
  status -- active | extended | completed | returned
)

-- Check-ins / actualizaciones
checkins (
  id, transit_id, user_id,
  note, photos[], is_public,
  vet_visit bool, vet_cost, vet_diagnosis,
  created_at
)

-- Chat
conversations (id, application_id, created_at)
messages (id, conversation_id, sender_id, body, created_at, read_at)

-- Reportes
reports (id, reporter_id, reported_user_id, reason, description, status, created_at)
```

---

## Fases de desarrollo

### Fase 1 — MVP (lanzamiento Santa Fe / Rosario)
- [ ] Auth con OAuth (Google, Facebook)
- [ ] Onboarding de roles
- [ ] Formulario de hogar completo para transitantes
- [ ] CRUD de mascotas para rescatistas/orgs
- [ ] Feed público de mascotas con filtros
- [ ] Perfil público de mascota (compartible)
- [ ] Flujo de postulación completo
- [ ] Chat básico en postulación
- [ ] Sistema de check-ins y seguimiento
- [ ] Alertas por check-in no respondido
- [ ] Flujo de cierre de tránsito (4 salidas)
- [ ] Panel superadmin básico

### Fase 2 — Crecimiento
- [ ] Expansión geográfica (CABA, Córdoba, Rosario ampliado)
- [ ] Donaciones a mascotas vía MercadoPago
- [ ] Timeline público mejorado (compartible en IG/FB)
- [ ] Notificaciones push (PWA)
- [ ] Dashboard de métricas para orgs

### Fase 3 — Escala
- [ ] App móvil nativa (React Native / Expo)
- [ ] Matching asistido por IA
- [ ] Red de veterinarios colaboradores
- [ ] Programa de subsidios para transitantes

---

## Workflow de desarrollo con agentes IA

- **Repositorio:** GitHub — `FrancoAmicone/PatitasEnEspera`
- **Ramas:** `main` (producción), `develop` (staging), feature branches por módulo
- **PR flow:** Los agentes crean feature branches → abren PR a `develop` → revisión automática → aprobación manual del dueño → merge a `main`
- **MCP configurado:** `@supabase/mcp-server-supabase` para que los agentes puedan interactuar con la DB directamente
- **CI/CD:** GitHub Actions en cada PR (lint, type check, build)
