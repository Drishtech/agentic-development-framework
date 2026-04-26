# Framework de Documentacion y Desarrollo Agentico

Este framework es agnostico de LLM y esta pensado para proyectos donde el diseno inicial se firma con intervencion humana, pero la implementacion posterior la ejecutan agentes autonomos con poca supervision. El objetivo es que cualquier agente pueda operar con contexto minimo sin contradecir decisiones previas.

## 0. Regla central

El repo es la memoria compartida. Todo agente debe poder operar leyendo:

1. `README.md`
2. `AGENTS.md`
3. `docs/CURRENT.md`
4. Su `SPRINT-N.md`
5. Sus `TASK-ID.md`
6. Los contratos y decisiones citados explicitamente por la tarea

Si una tarea requiere inferir contexto no citado, la tarea esta mal escrita y el Checker debe fallarla.

## A. Arbol de carpetas final

Se usan nombres ASCII (`design`, no `diseno`) para evitar problemas de tooling, shells y modelos pequenos. El contenido puede estar en espanol.

| Ruta | Proposito | Lectores | Escritor unico | Mutabilidad | Max tokens | Si se corrompe o se pierde |
|---|---|---:|---|---|---:|---|
| `README.md` | Indice navegable y explicacion en 90s | humano/planner/executor/checker | humano | editable controlado | 1200 | Se reconstruye desde `docs/CURRENT.md` y roadmap |
| `AGENTS.md` | Contrato obligatorio para cualquier LLM | planner/executor/checker | humano | inmutable post-firma salvo `CHANGE` | 1800 | Se bloquea trabajo hasta restaurarlo |
| `docs/` | Raiz documental | todos | humano | estructura estable | n/a | Se restaura desde git/backups |
| `docs/CURRENT.md` | Estado minimo vivo: fase, sprint activo, decisiones activas | todos | planner | editable append-resumido | 1500 | Se reconstruye desde sprint/decisiones |
| `docs/design/` | Diseno firmado de Fase 1 | humano/planner/checker | humano | inmutable post-firma | n/a | Se bloquean cambios mayores |
| `docs/design/DESIGN.md` | Diseno funcional y tecnico completo, dividido por secciones | humano/planner/checker | humano | inmutable post-firma | 3500 | Sprint no puede iniciar |
| `docs/design/SIGNOFF.md` | Firma humana de alcance, restricciones y aceptacion | humano/checker | humano | append-only | 1000 | No hay baseline valido |
| `docs/decisions/` | ADRs | planner/checker/executor si citado | planner | append-only | n/a | Se usa indice como recuperacion parcial |
| `docs/decisions/DECISION-NNN.md` | Decision arquitectonica atomica | todos si citado | planner | append-only; supersede, no editar | 1200 | Si esta citada, tarea bloqueada |
| `docs/decisions/INDEX.md` | Registro corto de decisiones activas/superadas | todos | checker | generado/validado | 1800 | Se regenera desde ADRs |
| `docs/guardrails/` | Reglas compactas derivadas de decisiones | planner/executor/checker | checker | generado/validado | n/a | Se regenera desde ADRs |
| `docs/guardrails/ARCHITECTURE_LOCKS.md` | Invariantes activas que ningun sprint puede contradecir | todos | checker | generado desde ADRs | 1500 | Executor no puede empezar |
| `docs/implementation/` | Planes ejecutables por sprint | planner/executor/checker | planner | append-only por sprint | n/a | Se restaura desde tareas |
| `docs/implementation/ROADMAP.md` | Fases previstas, no compromiso detallado | humano/planner | planner | editable por cambio aprobado | 1500 | No bloquea tareas activas |
| `docs/implementation/SPRINT-N.md` | Plan cerrado del sprint | todos | planner | inmutable tras handoff | 2200 | Sprint no se puede cerrar |
| `docs/tasks/` | Tareas atomicas ejecutables por modelos pequenos | executor/checker | planner | executor solo completa evidencias | n/a | Tarea bloqueada |
| `docs/tasks/TASK-ID.md` | Unidad de trabajo con contexto, TDD, paths, gates | executor/checker | planner | campos de evidencia editables | 2500 | No ejecutable |
| `docs/changes/` | Cambios sobre baseline firmado | humano/planner/checker | planner | append-only | n/a | Cambios mayores bloqueados |
| `docs/changes/CHANGE-NNN.md` | Solicitud/aprobacion de cambio menor/mayor/breaking | humano/planner/checker | planner | append-only con firma | 1200 | No se puede alterar diseno/ADR |
| `docs/testing/` | Estrategia y contratos de validacion | planner/executor/checker | checker | editable con cambio aprobado | n/a | Checker usa tests existentes y falla cobertura nueva |
| `docs/testing/TEST_STRATEGY.md` | Piramide, comandos, minimos por tipo de cambio | todos | checker | editable por `CHANGE` | 1500 | Sprint no puede cerrar |
| `docs/testing/ACCEPTANCE.md` | Criterios globales de aceptacion | humano/planner/checker | humano | inmutable post-firma | 1500 | Cierre bloqueado |
| `docs/checks/` | Evidencias de verificacion | checker | checker | append-only | n/a | Se rerunean checks |
| `docs/checks/CHECK-SPRINT-N.md` | Resultado del Checker | humano/checker | checker | append-only | 1500 | Sprint no cerrado |
| `scripts/verify_sprint.py` | Gate local verificable | checker/planner | checker | editable con test propio | 2000 | Cierre manual invalido |
| `scripts/verify_docs.py` | Valida enlaces, ADRs, cambios, locks | checker | checker | editable con test propio | 2000 | Drift no detectable |

## B. Plantillas reales y copiables

### `README.md`

```markdown
# {{PROJECT_NAME}}

## Que es
Sistema: {{PROJECT_NAME}}.
Usuario principal: {{PRIMARY_USER_ROLE}}.
Resultado observable: {{ONE_SENTENCE_OUTCOME}}.

## Estado actual
Fase activa: {{PHASE_NUMBER}}.
Sprint activo: {{SPRINT_NUMBER_OR_NONE}}.
Baseline firmado: `docs/design/SIGNOFF.md`.
Resumen vivo: `docs/CURRENT.md`.

## Leer en 90 segundos
1. `AGENTS.md` para reglas obligatorias.
2. `docs/CURRENT.md` para estado actual.
3. `docs/guardrails/ARCHITECTURE_LOCKS.md` para restricciones no negociables.
4. Sprint activo en `docs/implementation/SPRINT-{{NNN}}.md`.

## Mapa de documentacion
- Diseno firmado: `docs/design/DESIGN.md`
- Decisiones: `docs/decisions/INDEX.md`
- Implementacion: `docs/implementation/`
- Tareas: `docs/tasks/`
- Testing: `docs/testing/`
- Cambios aprobados: `docs/changes/`

## Comandos estandar
Instalar: `{{INSTALL_COMMAND}}`
Tests rapidos: `{{FAST_TEST_COMMAND}}`
Tests completos: `{{FULL_TEST_COMMAND}}`
Verificar sprint: `python scripts/verify_sprint.py docs/implementation/SPRINT-{{NNN}}.md`
```

### `AGENTS.md`

```markdown
# AGENTS.md

## Contrato obligatorio

Este repo se trabaja con Planner, Executor y Checker. Ningun agente puede saltarse su rol.

## Reglas no negociables

1. TDD red/green obligatorio para todo cambio de comportamiento.
2. YAGNI: no crear abstracciones, servicios, colas, adapters ni configuraciones no requeridas por una tarea.
3. DRY: si aparece la tercera repeticion semantica, detenerse y proponer refactor dentro de la tarea o abrir cambio.
4. Ningun agente contradice `docs/guardrails/ARCHITECTURE_LOCKS.md`.
5. Ningun agente edita documentos fuera de su permiso R/W/V.
6. Si falta contexto, el Executor debe marcar `BLOCKED_CONTEXT_MISSING`, no inferir.
7. Todo cambio de diseno firmado requiere `CHANGE-NNN.md`.
8. Todo cambio de decision activa requiere ADR nuevo que supersede al anterior.

## Lectura maxima por rol

Planner:
- Debe leer `README.md`, `AGENTS.md`, `docs/CURRENT.md`, `docs/decisions/INDEX.md`, `docs/guardrails/ARCHITECTURE_LOCKS.md`.
- Puede leer maximo 3 documentos de diseno adicionales por sprint.
- Puede leer maximo 2 sprints anteriores completos.

Executor:
- Debe leer `AGENTS.md`, `docs/CURRENT.md`, su `SPRINT-N.md`, su `TASK-ID.md`, y ADRs citadas.
- No debe leer el repo entero.
- No debe tocar `docs/design/`, `docs/decisions/`, `docs/changes/`.

Checker:
- Debe leer `AGENTS.md`, `SPRINT-N.md`, tareas del sprint, diff de archivos modificados, tests declarados y ADRs citadas.
- Valida, no implementa features.

## Regla de bloqueo

Usar exactamente una de estas marcas:
- `BLOCKED_CONTEXT_MISSING`
- `BLOCKED_DECISION_CONFLICT`
- `BLOCKED_TEST_UNCLEAR`
- `BLOCKED_SCOPE_TOO_LARGE`

Toda marca debe incluir archivo, seccion y pregunta concreta.
```

### `docs/design/DESIGN.md`

```markdown
# DESIGN.md

Status: SIGNED | DRAFT
Project: {{PROJECT_NAME}}
Design version: {{DESIGN_VERSION}}
Human owner: {{HUMAN_OWNER}}
Signed in: `docs/design/SIGNOFF.md`

## 1. Objetivo
Resultado de negocio: {{BUSINESS_OUTCOME}}
Usuarios incluidos: {{IN_SCOPE_USERS}}
Usuarios excluidos: {{OUT_OF_SCOPE_USERS}}

## 2. Alcance firmado
Incluido:
- {{IN_SCOPE_CAPABILITY_1}}
- {{IN_SCOPE_CAPABILITY_2}}

Excluido explicitamente:
- {{OUT_OF_SCOPE_CAPABILITY_1}}
- {{OUT_OF_SCOPE_CAPABILITY_2}}

## 3. Dominio
Entidades:
- {{ENTITY_NAME}}: campos obligatorios={{FIELDS}}, invariantes={{INVARIANTS}}

Reglas:
- {{RULE_ID}}: {{RULE_STATEMENT}}, fuente={{SOURCE}}

## 4. Interfaces
API/UI/CLI:
- Contract ID: {{CONTRACT_ID}}
- Operation: {{OPERATION_NAME}}
- Input valido: {{VALID_INPUT_SHAPE}}
- Output valido: {{VALID_OUTPUT_SHAPE}}
- Errores esperados: {{ERROR_CODES}}

## 5. Datos
Persistencia: {{PERSISTENCE_CHOICE}}
Migraciones requeridas: {{YES_NO}}
Datos sensibles: {{SENSITIVE_DATA_CLASSES}}

## 6. Restricciones
Rendimiento: {{PERFORMANCE_LIMIT}}
Seguridad: {{SECURITY_LIMIT}}
Operacion: {{OPERATIONS_LIMIT}}
Coste: {{COST_LIMIT}}

## 7. Testing aceptado
Comando minimo: {{FAST_TEST_COMMAND}}
Comando completo: {{FULL_TEST_COMMAND}}
Criterio de aceptacion global: `docs/testing/ACCEPTANCE.md`

## 8. Riesgos firmados
- Riesgo: {{RISK_NAME}}
  Probabilidad: LOW | MEDIUM | HIGH
  Impacto: LOW | MEDIUM | HIGH
  Mitigacion obligatoria: {{MITIGATION}}
```

### `docs/implementation/SPRINT-N.md`

```markdown
# SPRINT-{{NNN}}

Status: PLANNED | IN_PROGRESS | CHECKING | CLOSED | BLOCKED
Planner: {{PLANNER_ID}}
Checker: {{CHECKER_ID}}
Created: {{YYYY-MM-DD}}
Change class: MINOR | MAJOR | BREAKING
Authorized by: PLANNER | HUMAN

## Objetivo del sprint
Outcome verificable: {{OBSERVABLE_OUTCOME}}

## Lectura obligatoria del Executor
1. `AGENTS.md`
2. `docs/CURRENT.md`
3. `docs/guardrails/ARCHITECTURE_LOCKS.md`
4. Este archivo
5. Tarea asignada en `docs/tasks/`

## Decisiones activas relevantes
- {{DECISION_KEY}} -> `docs/decisions/DECISION-{{NNN}}.md`

## Tareas
- `docs/tasks/TASK-{{TASK_ID_1}}.md` owner={{EXECUTOR_ROLE}} status=READY
- `docs/tasks/TASK-{{TASK_ID_2}}.md` owner={{EXECUTOR_ROLE}} status=READY

## Paths permitidos
- {{ALLOWED_PATH_PATTERN_1}}
- {{ALLOWED_PATH_PATTERN_2}}

## Paths prohibidos
- `docs/design/**`
- `docs/decisions/**`
- `docs/changes/**`
- {{FORBIDDEN_PATH_PATTERN}}

## Comandos de validacion
Red test esperado antes de implementacion: {{RED_TEST_COMMAND}}
Green test minimo: {{FAST_TEST_COMMAND}}
Green test completo: {{FULL_TEST_COMMAND}}

## Handoff al Executor
El Executor puede empezar solo si todas las tareas estan en status `READY` y cada tarea tiene criterios de aceptacion, test rojo esperado, paths permitidos y ADRs relevantes.

## Handoff al Checker
El sprint pasa a `CHECKING` cuando todas las tareas estan en `IMPLEMENTED` y contienen evidencia red/green.
```

### `docs/tasks/TASK-ID.md`

````markdown
# TASK-{{ID}}

Status: READY | IN_PROGRESS | IMPLEMENTED | CHECK_FAILED | CLOSED | BLOCKED
Sprint: `docs/implementation/SPRINT-{{NNN}}.md`
Executor: {{EXECUTOR_ID_OR_ROLE}}
Change class: MINOR | MAJOR | BREAKING

## Objetivo atomico
Al terminar esta tarea, sera verdadero: {{SINGLE_VERIFIABLE_STATEMENT}}

## Contexto minimo
Leer solo:
- `AGENTS.md`
- `docs/CURRENT.md`
- `docs/guardrails/ARCHITECTURE_LOCKS.md`
- `docs/implementation/SPRINT-{{NNN}}.md`
- Esta tarea
- `docs/decisions/DECISION-{{NNN}}.md` porque {{DECISION_REASON}}

## Decisiones que toca
- Uses: {{DECISION_KEY}}
- Changes: NONE | `CHANGE-{{NNN}}.md`

## Archivos permitidos
- {{ALLOWED_FILE_OR_GLOB}}

## Archivos prohibidos
- `docs/design/**`
- `docs/decisions/**`
- `docs/changes/**`
- {{TASK_SPECIFIC_FORBIDDEN_GLOB}}

## TDD obligatorio
Test rojo que debe fallar primero:
- Archivo de test: {{TEST_FILE}}
- Nombre del test: {{TEST_NAME}}
- Comando: {{RED_TEST_COMMAND}}
- Motivo exacto del fallo esperado: {{EXPECTED_FAILURE_REASON}}

Evidencia red pegada por Executor:
```text
PENDING_RED_OUTPUT
```

Implementacion minima permitida:
- {{MINIMAL_IMPLEMENTATION_BOUNDARY}}

Tests green requeridos:
- {{FAST_TEST_COMMAND}}
- {{FULL_OR_TARGETED_TEST_COMMAND}}

Evidencia green pegada por Executor:
```text
PENDING_GREEN_OUTPUT
```

## Casos obligatorios
Happy path:
- {{HAPPY_PATH_ASSERTION}}

Edge cases:
- {{EDGE_CASE_1}}
- {{EDGE_CASE_2}}

Negative cases:
- {{NEGATIVE_CASE_1}}

## Anti-minimo-esfuerzo
El Executor debe completar:
- [ ] Existe test rojo antes de codigo productivo.
- [ ] El test falla por la razon esperada, no por error de sintaxis/config.
- [ ] Cada criterio de aceptacion tiene al menos una asercion.
- [ ] No se anadieron abstracciones fuera del objetivo atomico.
- [ ] No se duplico logica existente; si hay duplicacion, indicar archivo y justificacion.
- [ ] No se tocaron archivos prohibidos.
- [ ] No quedan `TODO`, `FIXME`, `PLACEHOLDER` ni skips nuevos.

## Criterios de aceptacion
- AC1: {{ASSERTABLE_ACCEPTANCE_CRITERION}}
- AC2: {{ASSERTABLE_ACCEPTANCE_CRITERION}}

## Notas del Executor
Archivos cambiados:
- PENDING_CHANGED_FILE_LIST

Decisiones tomadas dentro de la tarea:
- NONE

Bloqueos:
- NONE
````

### `docs/changes/CHANGE-NNN.md`

```markdown
# CHANGE-{{NNN}}

Status: PROPOSED | APPROVED | REJECTED | APPLIED
Class: MINOR | MAJOR | BREAKING
Requested by: HUMAN | PLANNER | CHECKER
Approver required: PLANNER | HUMAN
Approved by: {{APPROVER_ID}}
Date: {{YYYY-MM-DD}}

## Cambio solicitado
Cambiar desde: {{CURRENT_SIGNED_BEHAVIOR_OR_DECISION}}
Cambiar hacia: {{NEW_BEHAVIOR_OR_DECISION}}

## Motivo
Razon operativa: {{CONCRETE_REASON}}
Coste de no hacerlo: {{CONCRETE_RISK}}

## Impacto
Diseno firmado afectado:
- `docs/design/DESIGN.md#{{SECTION_ID}}` | NONE

Decisiones afectadas:
- `docs/decisions/DECISION-{{NNN}}.md` | NONE

Contratos afectados:
- {{CONTRACT_ID}} | NONE

Datos/migraciones:
- NONE | {{MIGRATION_ID}}

## Accion obligatoria
- [ ] Si cambia diseno firmado, actualizar version y `SIGNOFF.md`.
- [ ] Si cambia ADR activa, crear ADR nueva con `Supersedes`.
- [ ] Si cambia contrato publico, clasificar como BREAKING.
- [ ] Si cambia testing, actualizar `docs/testing/`.

## Resultado
Applied in sprint: `docs/implementation/SPRINT-{{NNN}}.md`
Checker report: `docs/checks/CHECK-SPRINT-{{NNN}}.md`
```

### `docs/decisions/DECISION-NNN.md`

```markdown
# DECISION-{{NNN}}: {{DECISION_TITLE}}

Status: PROPOSED | ACCEPTED | SUPERSEDED | REJECTED
Decision key: {{STABLE_KEY_UPPER_SNAKE}}
Date: {{YYYY-MM-DD}}
Owner: PLANNER
Supersedes: NONE | DECISION-{{NNN}}
Superseded by: NONE | DECISION-{{NNN}}
Change request: NONE | `docs/changes/CHANGE-{{NNN}}.md`

## Contexto
Problema concreto: {{PROBLEM_STATEMENT}}
Restricciones aplicables:
- {{CONSTRAINT}}

## Decision
Se decide: {{ONE_CLEAR_DECISION}}

## Consecuencias
Positivas:
- {{POSITIVE_CONSEQUENCE}}

Negativas aceptadas:
- {{NEGATIVE_CONSEQUENCE}}

## Invariante para ARCHITECTURE_LOCKS
Lock ID: {{LOCK_ID}}
Regla: {{NON_NEGOTIABLE_RULE}}
Aplica a paths:
- {{PATH_PATTERN}}

## Como validar
Checker debe verificar:
- {{VALIDATION_RULE_1}}
- {{VALIDATION_RULE_2}}

## Alternativas descartadas
- Alternativa: {{ALTERNATIVE_NAME}}
  Descartada porque: {{SPECIFIC_REASON}}
```

## C. Flujo Planner -> Executor -> Checker

### Planner

Lee maximo:

1. `README.md`
2. `AGENTS.md`
3. `docs/CURRENT.md`
4. `docs/guardrails/ARCHITECTURE_LOCKS.md`
5. `docs/decisions/INDEX.md`
6. Hasta 3 documentos de `docs/design/`
7. Hasta 2 sprints anteriores

Produce:

- `SPRINT-N.md`
- Uno o mas `TASK-ID.md`
- `CHANGE-NNN.md` si toca baseline firmado
- `DECISION-NNN.md` si crea/supersede arquitectura

No puede tocar:

- Codigo productivo
- Evidencia red/green
- Reportes de Checker

Handoff exacto:

> Cambiar `SPRINT-N.md` a `PLANNED` y todas sus tareas a `READY`. Si alguna tarea no tiene test rojo esperado, paths permitidos y decisiones relevantes, el handoff es invalido.

### Executor

Lee maximo:

1. `AGENTS.md`
2. `docs/CURRENT.md`
3. `docs/guardrails/ARCHITECTURE_LOCKS.md`
4. `SPRINT-N.md`
5. Su `TASK-ID.md`
6. ADRs citadas por la tarea
7. Archivos de codigo permitidos por la tarea

Produce:

- Test rojo
- Codigo minimo
- Test green
- Evidencia pegada en `TASK-ID.md`
- Lista de archivos cambiados

No puede tocar:

- `docs/design/**`
- `docs/decisions/**`
- `docs/changes/**`
- `docs/testing/**` salvo que la tarea lo permita explicitamente

Handoff exacto:

> Cambiar tarea a `IMPLEMENTED`, pegar salida red/green, listar archivos modificados y no dejar bloqueos abiertos.

### Checker

Lee maximo:

1. `AGENTS.md`
2. `SPRINT-N.md`
3. Tareas del sprint
4. `docs/guardrails/ARCHITECTURE_LOCKS.md`
5. ADRs citadas
6. Diff de archivos modificados
7. Tests declarados

Produce:

- `docs/checks/CHECK-SPRINT-N.md`
- Actualizacion de `docs/CURRENT.md`
- Regeneracion/validacion de `docs/decisions/INDEX.md`
- Regeneracion/validacion de `ARCHITECTURE_LOCKS.md`

No puede tocar:

- Codigo productivo, salvo que el usuario pida explicitamente fix de CI
- Diseno firmado

Handoff exacto:

> Si todos los gates pasan, marcar sprint `CLOSED`, tareas `CLOSED`, escribir check report y actualizar `CURRENT.md`.

## D. Definicion ejecutable de "sprint cerrado"

Un sprint esta cerrado solo si este comando termina con exit code `0`:

```bash
python scripts/verify_sprint.py docs/implementation/SPRINT-005.md
```

Condiciones verificables:

1. `SPRINT-N.md` existe y status=`CHECKING` o `CLOSED`.
2. Todas las tareas referenciadas existen.
3. Cada tarea tiene status=`IMPLEMENTED` o `CLOSED`.
4. Cada tarea contiene evidencia `RED_OUTPUT` distinta de `PENDING_RED_OUTPUT`.
5. Cada tarea contiene evidencia `GREEN_OUTPUT` distinta de `PENDING_GREEN_OUTPUT`.
6. Ninguna tarea contiene `TODO`, `FIXME`, `PLACEHOLDER`, `PENDING_`.
7. Todo archivo modificado coincide con `allowed paths`.
8. Ningun archivo modificado coincide con `forbidden paths`.
9. Toda `Decision key` existe en `docs/decisions/INDEX.md`.
10. Ninguna tarea cambia una decision sin `CHANGE-NNN.md` aprobado.
11. Si hay `CHANGE` mayor/breaking, tiene aprobacion humana.
12. Tests declarados fueron ejecutados y su salida esta registrada.
13. `docs/checks/CHECK-SPRINT-N.md` existe con verdict=`PASS`.
14. `docs/CURRENT.md` referencia el sprint cerrado.
15. `ARCHITECTURE_LOCKS.md` esta sincronizado con ADRs aceptadas.

## E. Mecanismo anti-drift

No basta con "usar ADRs". El mecanismo tiene tres capas.

### 1. Decision key estable

Cada ADR aceptada define una `Decision key`, por ejemplo:

```text
EMAIL_PROVIDER_SENDGRID
AUTH_SESSION_JWT
DB_POSTGRES_PRIMARY
```

Las tareas no citan narrativa larga; citan keys.

### 2. Indice corto obligatorio

`docs/decisions/INDEX.md` contiene solo decisiones activas:

```markdown
# Decision Index

- key: EMAIL_PROVIDER_SENDGRID
  status: ACCEPTED
  adr: docs/decisions/DECISION-004.md
  lock: LOCK-EMAIL-001
  applies_to: src/email/**, config/email/**
  superseded_by: NONE
```

Esto permite que el sprint 12 lea un archivo corto y no los sprints 1-11.

### 3. Architecture locks generados

`ARCHITECTURE_LOCKS.md` traduce ADRs a reglas ejecutables por revision:

```markdown
# Architecture Locks

## LOCK-EMAIL-001
Decision key: EMAIL_PROVIDER_SENDGRID
Rule: Production email delivery must use SendGrid through the email adapter boundary.
Applies to:
- `src/email/**`
- `config/email/**`
Checker validation:
- No direct AWS SES client in production email paths.
- Email provider config uses SENDGRID_API_KEY.
```

Regla anti-drift:

> Si una tarea toca un path cubierto por un lock, debe citar su decision key. Si contradice el lock, debe tener `CHANGE-NNN.md` aprobado y ADR nueva con `Supersedes`.

## F. Mecanismo anti-minimo-esfuerzo

El framework fuerza comportamiento mediante gates, no buena voluntad:

1. TDD red/green esta en `TASK-ID.md` y `verify_sprint.py` falla si falta evidencia.
2. Cada aceptacion tiene que mapearse a test o asercion.
3. El Executor no puede escribir "he probado manualmente" como sustituto del comando.
4. El Checker rechaza tareas con `PENDING_`, `TODO`, skips nuevos o archivos prohibidos.
5. El Planner debe escribir edge cases y negative cases; si no existen, la tarea no esta `READY`.
6. YAGNI se aplica con "implementacion minima permitida": si el Executor anade infraestructura no listada, falla.
7. DRY se aplica con declaracion obligatoria de duplicacion; si introduce tercera repeticion sin refactor o justificacion, falla.
8. Los paths permitidos evitan que el agente "arregle" el problema tocando zonas no autorizadas.
9. Los locks evitan contradicciones silenciosas con arquitectura previa.

## G. Matriz R/W/V por documento y rol

| Documento | Humano | Planner | Executor | Checker |
|---|---|---|---|---|
| `README.md` | R/W/V | R | R | V |
| `AGENTS.md` | R/W/V | R | R | V |
| `docs/CURRENT.md` | R | W | R | W/V |
| `docs/design/DESIGN.md` | R/W/V | R | R si citado | V |
| `docs/design/SIGNOFF.md` | W/V | R | R | V |
| `docs/decisions/DECISION-NNN.md` | R/V | W | R si citado | V |
| `docs/decisions/INDEX.md` | R | R | R | W/V |
| `docs/guardrails/ARCHITECTURE_LOCKS.md` | R | R | R | W/V |
| `docs/implementation/ROADMAP.md` | V | W | R | V |
| `docs/implementation/SPRINT-N.md` | R/V | W | R | V |
| `docs/tasks/TASK-ID.md` | R | W | W solo evidencias | V |
| `docs/changes/CHANGE-NNN.md` | W/V si mayor | W | R si citado | V |
| `docs/testing/TEST_STRATEGY.md` | R/V | R | R | W/V |
| `docs/testing/ACCEPTANCE.md` | W/V | R | R | V |
| `docs/checks/CHECK-SPRINT-N.md` | R | R | R | W |
| `scripts/verify_sprint.py` | R | R | R | W/V |

## H. Clasificacion de cambios

### Menor

Definicion:

- No cambia contrato publico.
- No cambia datos persistidos.
- No cambia ADR activa.
- No cambia alcance firmado.
- Anade o corrige comportamiento interno esperado.

Autoriza: Planner.

Accion:

- Crear sprint/tareas.
- No requiere `CHANGE-NNN.md`, salvo que toque texto firmado.

### Mayor

Definicion:

- Anade capacidad nueva.
- Anade integracion.
- Anade tabla/campo persistido compatible.
- Cambia estrategia de testing.
- Supersede una ADR sin romper contrato publico.

Autoriza: Humano.

Accion:

- Crear `CHANGE-NNN.md`.
- Crear o superseder ADR si aplica.
- Actualizar roadmap y `CURRENT.md`.

### Breaking

Definicion:

- Cambia API publica.
- Rompe compatibilidad de datos.
- Elimina comportamiento firmado.
- Cambia auth, permisos, proveedor critico o contrato operativo.
- Requiere migracion no reversible.

Autoriza: Humano con firma explicita.

Accion:

- `CHANGE-NNN.md` aprobado.
- Nueva version de diseno o anexo firmado.
- ADR nueva con `Supersedes`.
- Plan de migracion y rollback.
- Tests de compatibilidad o ruptura esperada.

## Casos de prueba obligatorios

### Caso 1 - Fase 1 desde cero

Proyecto: API REST de gestion de inventario para una tienda pequena.

#### Documentos al final de Fase 1, antes de codigo

```text
inventory-api/
├── README.md
├── AGENTS.md
└── docs/
    ├── CURRENT.md
    ├── design/
    │   ├── DESIGN.md
    │   └── SIGNOFF.md
    ├── decisions/
    │   ├── DECISION-001.md
    │   ├── DECISION-002.md
    │   └── INDEX.md
    ├── guardrails/
    │   └── ARCHITECTURE_LOCKS.md
    ├── implementation/
    │   └── ROADMAP.md
    └── testing/
        ├── TEST_STRATEGY.md
        └── ACCEPTANCE.md
```

#### Contenido firmado por humano

El humano firma:

- `docs/design/DESIGN.md`
- `docs/design/SIGNOFF.md`
- `docs/testing/ACCEPTANCE.md`
- `AGENTS.md`

Ejemplo de decisiones iniciales:

- `DECISION-001`: usar REST JSON con versionado `/v1`.
- `DECISION-002`: usar PostgreSQL como base primaria.
- `DECISION-003`: autenticacion JWT para endpoints privados.

#### Que leera el primer agente y en que orden

Primer Planner:

1. `README.md`
2. `AGENTS.md`
3. `docs/CURRENT.md`
4. `docs/design/DESIGN.md`
5. `docs/testing/ACCEPTANCE.md`
6. `docs/decisions/INDEX.md`
7. `docs/guardrails/ARCHITECTURE_LOCKS.md`

Primer Executor:

1. `AGENTS.md`
2. `docs/CURRENT.md`
3. `docs/implementation/SPRINT-001.md`
4. Su `docs/tasks/TASK-INV-001.md`
5. ADRs citadas: por ejemplo `DECISION-001`, `DECISION-002`

No lee todo el diseno si la tarea ya extrae el contexto necesario.

### Caso 2 - Sprint 5: alertas de stock bajo por email

Estado previo: auth, productos y reportes basicos ya implementados.

Feature: enviar alertas de stock bajo por email.

#### Que lee el Planner y por que solo eso

Planner lee:

1. `README.md`: confirmar mapa y comandos.
2. `AGENTS.md`: reglas de rol.
3. `docs/CURRENT.md`: estado actual y sprint anterior.
4. `docs/decisions/INDEX.md`: detectar decisiones activas.
5. `docs/guardrails/ARCHITECTURE_LOCKS.md`: ver restricciones.
6. `docs/design/DESIGN.md#productos`: entender stock.
7. `docs/design/DESIGN.md#notificaciones` si existe; si no existe, crea cambio mayor.
8. `docs/testing/ACCEPTANCE.md`: criterios globales.

Solo esos porque la feature toca stock, email y testing; no necesita leer auth completa ni reportes completos si sus contratos estan indexados.

#### Que produce el Planner

Produce:

- `docs/changes/CHANGE-005.md`, clase `MAJOR`, porque agrega integracion email.
- `docs/decisions/DECISION-004.md`, key `EMAIL_PROVIDER_SENDGRID`.
- Actualizacion validada de `docs/decisions/INDEX.md`.
- Actualizacion validada de `docs/guardrails/ARCHITECTURE_LOCKS.md`.
- `docs/implementation/SPRINT-005.md`.
- `docs/tasks/TASK-EMAIL-001.md`: adapter de email con test rojo.
- `docs/tasks/TASK-STOCK-002.md`: deteccion de stock bajo.
- `docs/tasks/TASK-ALERT-003.md`: integracion evento stock bajo -> email.
- `docs/tasks/TASK-CHECK-004.md`: pruebas de no duplicar alertas.

#### Que lee el Executor modelo pequeno

Para `TASK-ALERT-003.md` lee:

1. `AGENTS.md`
2. `docs/CURRENT.md`
3. `docs/guardrails/ARCHITECTURE_LOCKS.md`
4. `docs/implementation/SPRINT-005.md`
5. `docs/tasks/TASK-ALERT-003.md`
6. `docs/decisions/DECISION-004.md`
7. Archivos permitidos:
   - `src/stock/**`
   - `src/email/**`
   - `tests/stock/**`
   - `tests/email/**`

No lee auth, reportes ni sprints 1-4 completos.

#### Que pasa si en sprint 9 quieren migrar SendGrid a AWS SES

No se edita `DECISION-004`.

Se crea:

- `docs/changes/CHANGE-009.md`, clase `MAJOR` si la interfaz interna se mantiene; `BREAKING` si cambia contrato publico/configuracion operativa.
- `docs/decisions/DECISION-012.md`, key `EMAIL_PROVIDER_AWS_SES`, con `Supersedes: DECISION-004`.
- `DECISION-004` pasa a `SUPERSEDED`.
- `docs/decisions/INDEX.md` reemplaza key activa.
- `ARCHITECTURE_LOCKS.md` cambia de "no AWS SES" a "usar AWS SES via adapter".

Checker de sprint 9 falla si queda codigo productivo usando SendGrid fuera de una migracion explicita o si una tarea usa la key antigua.

#### Como el Checker valida sin leer el repo entero

Checker lee:

1. `SPRINT-005.md`
2. Tareas del sprint
3. Diff de archivos modificados
4. `DECISION-004.md`
5. `ARCHITECTURE_LOCKS.md`
6. Tests declarados

Valida:

- Hay test rojo para stock bajo.
- Hay test rojo para envio email.
- No se envian emails duplicados para la misma condicion.
- El proveedor email aparece detras del adapter.
- No hay cliente SendGrid fuera de `src/email/**`.
- Los comandos green estan pegados.
- Los paths tocados son los permitidos.

## Auto-critica obligatoria

1. El framework depende de disciplina documental local. Se rompe si el equipo acepta merges sin ejecutar `verify_sprint.py`; en ese escenario, los locks existen pero no protegen nada.

2. El limite de 4000 tokens por archivo obliga a resumir. Se rompe en dominios regulados o cientificos donde una decision necesita mucha evidencia; solucion parcial: partir ADRs por decision y mover evidencia larga a anexos no obligatorios.

3. La validacion anti-drift no entiende semantica profunda del codigo. Se rompe si un agente contradice una decision con una abstraccion indirecta que no coincide con paths o patrones; mitigacion: locks con paths precisos, tests de arquitectura y revision humana para cambios mayores.

## Referencias

- Diataxis: tomo la separacion por tipo de necesidad del lector; descarto documentacion extensa que obligue al Executor a leer tutoriales.
- AGENTS.md: tomo el contrato operativo en raiz del repo; descarto instrucciones largas no verificables.
- Karpathy LLM Wiki: tomo la idea de contexto pequeno, explicito y orientado a agentes; descarto depender de razonamiento implicito del modelo.
- ADRs de Michael Nygard: tomo decisiones append-only con consecuencias; descarto ADRs como archivo historico pasivo sin locks ni validacion.
