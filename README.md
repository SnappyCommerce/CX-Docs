# 🧭 CX-Docs

**Qué necesita cada cliente y cómo comprobamos que el bot lo cumple.**

Este repo es la fuente de verdad del equipo de CX de SnappyCommerce. Cada marca tiene:

1. 📄 su **alcance funcional**: qué tiene que hacer el agente de IA, en qué casos y con qué reglas.
2. 🧪 sus **tests**: las conversaciones que corremos contra el bot para comprobar que cada requisito se cumple.

> Si no está en el alcance, no es un requisito. Si no tiene test, no está verificado.

---

## 📁 Estructura

```
Clientes/
└── <Marca>/
    ├── Owners <Marca>.md              ← responsables (Account Owner y CX, por Snappy ID)
    ├── Alcance Funcional <Marca>.md   ← qué necesita el cliente
    └── Tests <Marca>.json             ← cómo verificamos cada necesidad
```

- La carpeta de cada cliente usa el nombre de la marca (por ejemplo `Clientes/Naldo/`).
- El alcance se llama siempre `Alcance Funcional <Marca>.md`, y los tests `Tests <Marca>.json` (un solo archivo por marca).
- La primera línea del alcance (`> Fuente: …`) apunta al documento original en Drive.

## 🔁 Flujo de trabajo

```
 Drive (doc del cliente)
        │  convertir a .md
        ▼
 Alcance Funcional <Marca>.md ──► requisitos
        │  crear tests
        ▼
 Tests <Marca>.json ──► correr contra el bot ──► reporte de QA
        ▲                                     │
        └──────── corregir alcance / bot ◄────┘
```

1. **Alcance:** cuando el cliente cierra o cambia su alcance en Drive, se actualiza el `.md` acá, en un PR.
2. **Tests:** cada requisito del alcance tiene al menos un test conversacional. Si el alcance cambia, los tests cambian en el mismo PR.
3. **QA:** los tests se corren contra el bot de la tienda y el resultado se compara con la corrida anterior.

### Con Claude Code

| Querés… | Decí |
|---|---|
| Crear o actualizar los tests de una marca | `creá los tests de Naldo` |
| Correr los tests de una marca | `corré los tests de Naldo` |
| Hacer QA exploratorio de una tienda | `hagamos QA de Naldo` |

## ✍️ Cómo contribuir

- Trabajá en una rama y abrí un PR. No pushees directo a `master`.
- Un PR por cliente y por cambio (por ejemplo, "Naldo: agrega flujo de cambios").
- Mantené el alcance fiel al documento del cliente. Si algo está pendiente o es ambiguo, marcalo como **Pendiente** en vez de inventarlo.
- Cuando cambia un requisito, revisá qué tests lo cubren y actualizalos.

## 🔒 Confidencialidad

Este repo tiene información de clientes. Es privado: no copies su contenido fuera de Snappy ni subas credenciales, tokens ni datos personales de clientes finales.
