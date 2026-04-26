# Agentic Development Framework (ADF)

Un framework agnóstico de LLM diseñado para estructurar el desarrollo de software ejecutado por agentes autónomos (especialmente modelos de 8B-32B). Su objetivo es garantizar código *production-ready* mediante el control estricto del contexto, TDD forzado y validación mecánica, minimizando la intervención humana.

## 🎯 Objetivo

Los agentes LLM actuales sufren de memoria a corto plazo, pérdida de contexto (*context drift*) y tendencia a la pereza técnica (evitar refactorización y tests). 

ADF resuelve esto serializando la intención humana en artefactos de texto rígidamente estructurados. No confía en la "buena voluntad" del agente, sino en **gates mecánicos y ejecutables** (`scripts/verify_sprint.py`) que bloquean el avance si el agente no aporta evidencia de TDD (Red/Green) o viola invariantes de arquitectura.

## 🏗️ Inspiración y Bases Teóricas

Este framework no nace del vacío. Toma patrones de la ingeniería de software tradicional y los adapta a las limitaciones cognitivas de los LLMs:

1. **[Diátaxis](https://diataxis.fr/)**: Se adopta la separación estricta de la documentación por propósito, eliminando tutoriales innecesarios para obligar al agente a operar solo con manuales de referencia (Referencia/How-to).
2. **[Architecture Decision Records (ADRs) de Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)**: Implementado mediante inmutabilidad y *superseding*. Se extiende creando **Architecture Locks**: reglas ejecutables extraídas de los ADRs para que los agentes las respeten sin tener que leer el historial completo.
3. **[Patrones de Context Engineering (Karpathy LLM Wiki)](https://github.com/karpathy/LLM-Wiki)**: Se materializa limitando drásticamente los tokens que cada rol (Planner, Executor, Checker) tiene permitido leer, evitando la saturación del contexto.

## ⚙️ Mecanismos Core

- **Anti-drift**: Uso de *Decision keys* estables e índices cortos (`ARCHITECTURE_LOCKS.md`) que el agente debe respetar al tocar *paths* específicos del repositorio.
- **Anti-mínimo-esfuerzo**: El agente no puede cerrar una tarea sin aportar evidencia en consola (pegada en el markdown) de que un test falló (RED) por la razón correcta antes de escribir la implementación funcional (GREEN).
- **Handoffs Deterministas**: El sistema se basa en 3 roles (Planner, Executor, Checker) con matrices de permisos de lectura/escritura (R/W/V) inquebrantables.

## 📖 Especificación Completa

Lee el documento completo del framework y sus plantillas operacionales aquí:
👉 **[Especificación del Framework (framework-spec.md)](./framework-spec.md)**

## 🤝 Llamado a la Comunidad (Request for Comments)

Este framework está en fase de propuesta y busco *feedback* de profesionales operando con agentes LLM en entornos reales. Las áreas principales de debate son:

- **Límites de tokens**: ¿Son realistas los umbrales de lectura estipulados para modelos locales de 32B?
- **Mecanismos de bloqueo**: ¿Existen vectores de evasión donde un agente pueda saltarse el `verify_sprint.py` alterando la salida del test?
- **Casos límite**: Experiencias intentando forzar refactorizaciones estéticas o de UI bajo este nivel de TDD estricto.

Abre un *Issue* para discutir la especificación o envía un PR si tienes mejoras para los scripts de validación.
