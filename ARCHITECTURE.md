# Arquitectura del Sistema
┌─────────────────────────────────────┐
│             YOUR NETWORK            │
├─────────────────────────────────────┤
│                                     │
│   ┌──────────────┐  ┌────────────┐  │
│   │    NetBox    │  │  Oxidized  │  │
│   │   (Docker)   │◄─│  (Docker)  │  │
│   └──────────────┘  └────────────┘  │
│        ▲                   │        │
│        │ API               │ SSH    │
│        └─────────┬─────────┘        │
│                  │                  │
│    ┌─────────────▼──────────────┐   │
│    │      Network devices       │   │
│    │       - Switch 1           │   │
│    │       - Switch 2           │   │
│    │       - Router             │   │
│    └────────────────────────────┘   │
│                                     │
│  ┌────────────────────────────────┐ │
│  │   /backups/oxidized/ (volume)  │ │
│  │      Saved configurations      │ │
│  └────────────────────────────────┘ │
└─────────────────────────────────────┘

### NetBox
- Base de datos de inventario
- Descubre dispositivos automáticamente
- API REST

### Oxidized
- Lee de NetBox via API
- Ejecuta backups cada X tiempo
- Guarda en volumen persistente

### Docker
- Contenedores aislados
- Volúmenes para persistencia
- Red interna

## Flujo de datos
[Diagrama ASCII o imagen]

NetBox API
   ↓
Oxidized lee dispositivos
   ↓
SSH a cada dispositivo
   ↓
Guarda config en /backups/
   ↓
Versionado con Git