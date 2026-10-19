# Plan de Traspaso del Proyecto pdf-summarizer a BIREME

**Estado del proyecto al inicio del traspaso**: v0.12.0 · 16 fases completadas
(eval-specs fase0–fase15) · CI verde en `master` · PR #6 (observabilidad
durable, fork BIREME) en revisión.

**Objetivo**: transferir la propiedad, gestión y evolución del proyecto a la
célula de BIREME de forma que en ~6 semanas el equipo opere con total
autonomía, preservando la disciplina de calidad (EDD + arquitectura hexagonal)
que sostiene el proyecto.

**Roles**:

| Rol | Quién | Responsabilidad durante el traspaso |
|---|---|---|
| Cedente | @idourra | Transferencia, walkthroughs, review en transición |
| Maintainers entrantes | 2+ personas designadas por BIREME | Merge, releases, gobernanza |
| Contributors | Resto de la célula | Desarrollo vía PR |

> ⚠️ Requisito previo: BIREME designa **mínimo 2 maintainers** con cuenta
> GitHub antes de la Fase 1. Un solo admin = bus factor 1, y eso es
> exactamente lo que este plan elimina.

---

## Fase 0 — Cierre de pendientes (semana 1)

Objetivo: no traspasar trabajo a medias.

- [ ] Merge del PR #6 (observabilidad durable + optimización OCR + suite
      reforzada) una vez resuelto el lint de `abstracts.py` y documentado el
      breaking change de exit codes en `CHANGELOG.md`.
- [ ] Limpiar ramas remotas ya mergeadas (`feat/bibframe-export`,
      `feat/docker-cli-wrapper`, `fix/ci-uv-cache-infra`,
      `fix/docker-ollama-runtime-deps`).
- [ ] Revisar issues abiertos: cerrar obsoletos, etiquetar los vigentes.
- [ ] Publicar release `v0.13.0` post-merge del PR #6 (incluye el salto a
      `report.json` 3.0 y el cambio de exit codes) como línea base del
      traspaso.
- [ ] Verificación de instalación limpia: seguir `INSTALL.md` en una máquina
      sin estado previo, hecho por alguien de BIREME (no por el cedente).
      Todo hueco encontrado se corrige en `INSTALL.md`, no por chat.

**Criterio de salida**: `master` en verde, sin PRs abiertos del cedente, con
release taggeada, y un miembro de BIREME ha instalado desde cero sin ayuda.

---

## Fase 1 — Transferencia de propiedad y gobernanza técnica (semana 2)

Objetivo: la organización BIREME es dueña del repo y las reglas del proceso
quedan aplicadas por GitHub, no por costumbre.

### 1.1 Transferencia del repositorio

- [ ] Transferir `idourra/pdf-summarizer` a la organización GitHub de BIREME
      (Settings → General → Transfer ownership). GitHub mantiene historial,
      issues, PRs, releases y crea redirects de los clones existentes.
- [ ] El fork `bireme/pdf-summarizer` actual se archiva o elimina tras la
      transferencia (evita confusión de "cuál es el canónico").
- [ ] Actualizar remotes en las máquinas del equipo:
      `git remote set-url origin git@github.com:<org-bireme>/pdf-summarizer.git`
- [ ] Actualizar URLs en `README.md`, `pyproject.toml` (`[project.urls]`),
      `COLABORADORES-INVITAR.md` y `GITHUB-TEAM-WELCOME.md` (o retirar estos
      dos últimos si dejan de aplicar).

### 1.2 Equipos y permisos

- [ ] Team `pdfsum-maintainers` (2+ personas): rol *Maintain* + capacidad de
      merge y release.
- [ ] Team `pdfsum-contributors`: rol *Write* (push de ramas, no merge a
      `master`).
- [ ] @idourra queda como *Write* durante la transición (Fases 3–4) y se
      retira en la Fase 5.

### 1.3 Protección de `master` (la política EDD como regla, no como costumbre)

- [ ] Branch protection en `master`:
  - Require pull request (mínimo 1 review aprobada).
  - Require status checks: `test (3.10)`, `test (3.11)`, `test (3.12)`,
    `build`, `docker`, `docs`.
  - Prohibir push directo y force-push (incluidos admins).
  - Require branches up to date before merge.
- [ ] `CODEOWNERS` en la raíz:

  ```
  # Dominio puro: revisión de maintainer obligatoria
  /src/pdfsum/*.py          @<org-bireme>/pdfsum-maintainers
  /evals/                   @<org-bireme>/pdfsum-maintainers
  # Adapters e infraestructura
  /src/pdfsum/adapters/     @<org-bireme>/pdfsum-contributors
  ```

### 1.4 Secrets y publicación

- [ ] Auditar secrets de Actions usados por `ci.yml` y `publish.yml`;
      regenerarlos bajo cuentas/tokens de la organización BIREME.
- [ ] Decidir destino de publicación (PyPI/registry Docker propio de BIREME)
      y transferir o recrear las credenciales. Ningún token personal del
      cedente sobrevive a esta fase.

**Criterio de salida**: nadie (ni admins) puede pushear directo a `master`;
un PR de prueba demuestra que el flujo protegido funciona; `publish.yml`
ejecuta con credenciales de la org.

---

## Fase 2 — Institucionalización del proceso (semanas 2–3, en paralelo)

Objetivo: que la disciplina EDD + hexagonal sea un contrato escrito y
ejecutable que sobreviva a las personas.

### 2.1 Documentos de gobernanza (nuevos)

- [ ] **`GOVERNANCE.md`**: quién decide qué (maintainers deciden merge y
      releases por acuerdo simple; cambios de arquitectura requieren ADR),
      cómo se nombran/retiran maintainers, cadencia de releases.
- [ ] **`docs/adr/`**: registro de Architecture Decision Records. Migrar como
      ADR-001 la regla existente "el dominio no importa adaptadores" (hoy
      verificada por `tests/test_architecture.py`). Toda decisión
      arquitectónica futura entra como ADR numerado.

### 2.2 CONTRIBUTING.md reforzado

- [ ] Regla dura documentada: **toda feature/fix lleva su
      `evals/eval-spec-*.yaml` (o `-lite-`) antes del código**, con criterios
      que mapean 1:1 a tests (unit / integration / architecture / metrics).
- [ ] Flujo obligatorio: rama → eval-spec → tests → código → PR → CI verde →
      merge. Prohibido `--no-verify` y saltarse gates.
- [ ] Documentar el runner de suites (`tests/run_suite.py` del PR #6) y
      cuándo usar cada suite (unit, integration, contract, architecture,
      e2e, performance).

### 2.3 Plantillas

- [ ] **`.github/PULL_REQUEST_TEMPLATE.md`** con checklist:
  - [ ] Eval-spec asociado (ruta) o justificación de por qué no aplica.
  - [ ] Evidencia: lint + suite completa en verde (pegar resumen).
  - [ ] ¿Cambia `report.json`, la CLI o los exit codes? → documentado en
        `CHANGELOG.md` como breaking/no-breaking.
  - [ ] ¿Toca el dominio puro? → sin imports de adapters
        (`test_architecture.py` en verde).
- [ ] **Plantillas de issue**: bug (con versión, comando exacto, `report.json`
      adjunto) y feature (con criterio de aceptación esbozado, germen del
      futuro eval-spec).

### 2.4 Contratos estables

- [ ] Documentar `report.json` v3.0 como **contrato público** en
      `docs/CONTRATO-REPORT.md`: claves garantizadas, política de versionado
      (aditivo = minor, ruptura = major + `report_version` bump).
- [ ] Documentar exit codes de la CLI (`0` éxito, `1` fallos de
      procesamiento, `2` uso incorrecto) en `GUIA-USO.md`.

**Criterio de salida**: un PR nuevo de cualquier miembro sigue el proceso
completo solo leyendo `CONTRIBUTING.md` + plantilla, sin preguntar al cedente.

---

## Fase 3 — Transferencia de conocimiento (semanas 3–4)

Objetivo: traspasar criterio, no solo código.

### 3.1 Walkthroughs (sesiones grabadas, 60–90 min cada una)

- [ ] **Sesión 1 — Arquitectura**: hexagonal + DDD en este repo; dominio puro
      (`segment.py`, `qa.py`, `abstracts.py`, `pipeline.py`) vs adapters
      (`hybrid_ocr.py`, `pdf_batch.py`, `api_server.py`, cloud); por qué el
      test de arquitectura existe y cómo falla.
- [ ] **Sesión 2 — Pipeline y evals**: flujo PDF → OCR híbrido → segmentación
      → resumen jerárquico → QA → export (LILACS/BIBFRAME); anatomía de un
      eval-spec y su mapeo a tests; cómo se añadió una fase real (usar
      fase15-bibframe como ejemplo).
- [ ] **Sesión 3 — Operación**: Docker + Ollama, `bin/pdfsum-docker`,
      backends cloud, la nueva observabilidad del PR #6 (`events.jsonl`,
      `infrastructure.jsonl`, `run_id`), releases y `publish.yml`.
- [ ] Grabaciones enlazadas desde el README (sección "Onboarding").

### 3.2 Documento de estado y rumbo

- [ ] **`docs/ESTADO-Y-RUMBO.md`** escrito por el cedente:
  - Qué está completo (fase0–fase15, con una línea por fase).
  - Deuda técnica conocida y riesgos (con severidad).
  - Los 3–5 próximos pasos que el cedente haría, con justificación —
    candidatos: estabilizar contrato `report.json` 3.0, corpus de
    validación real para la heurística de abstracts (RESUME francés),
    mutation testing en CI (`mutmut` ya configurado en el PR #6),
    empaquetado/distribución institucional.
- [ ] Backlog priorizado como **milestone "Célula BIREME — Q1"** con esos
      próximos pasos convertidos en issues.

### 3.3 Pairing dirigido

- [ ] Cada maintainer entrante lidera **un ciclo completo** (eval-spec →
      tests → código → PR → merge) de un issue pequeño del milestone, con el
      cedente solo como reviewer. Esto valida que el proceso está
      efectivamente traspasado.

**Criterio de salida**: 3 sesiones grabadas y enlazadas; `ESTADO-Y-RUMBO.md`
mergeado; al menos 2 ciclos EDD completos ejecutados por maintainers de
BIREME sin intervención del cedente en el código.

---

## Fase 4 — Transición supervisada (semanas 4–6)

Objetivo: invertir los roles de forma controlada.

- [ ] El cedente **no escribe código**: solo revisa PRs (segundo par de
      ojos, sin ser bloqueante — la review requerida la dan los maintainers
      de BIREME).
- [ ] Los maintainers hacen **una release completa** (versionado, CHANGELOG,
      tag, publish) sin participación del cedente.
- [ ] Simulacro de incidencia: tomar un bug real o inyectado, y que el
      equipo lo lleve de issue a fix mergeado + release patch, midiendo
      dónde se atascan. Los atascos se convierten en mejoras de docs.
- [ ] Retro de traspaso a mitad de fase: ¿qué conocimiento sigue faltando?
      Sesión extra de walkthrough si hace falta.

**Criterio de salida**: una release publicada y un bug resuelto
end-to-end por BIREME sin el cedente.

---

## Fase 5 — Salida (fin de semana 6)

- [ ] Retirar permisos de escritura de @idourra (queda, si ambas partes
      quieren, como colaborador externo que contribuye por fork+PR como
      cualquiera).
- [ ] Verificación final de accesos: `gh api` de collaborators y teams —
      solo cuentas de BIREME con permisos; ningún secret personal activo.
- [ ] Anuncio en README: mantenimiento a cargo de la célula BIREME, canal de
      contacto del equipo (no del cedente).
- [ ] Acordar (opcional) una ventana de consultoría puntual de 1–3 meses
      para dudas de arquitectura, con canal y expectativas explícitas.

---

## Resumen de artefactos a producir

| Artefacto | Fase | Responsable |
|---|---|---|
| Release v0.13.0 (línea base) | 0 | Cedente |
| Transferencia repo + teams + branch protection + CODEOWNERS | 1 | Cedente + maintainers |
| Secrets de org (CI/publish) | 1 | Maintainers |
| `GOVERNANCE.md`, `docs/adr/ADR-001` | 2 | Cedente (draft) + maintainers (aprueban) |
| `CONTRIBUTING.md` reforzado + plantillas PR/issue | 2 | Cedente (draft) |
| `docs/CONTRATO-REPORT.md` (report.json 3.0) | 2 | Maintainers (autores del PR #6) |
| 3 walkthroughs grabados | 3 | Cedente |
| `docs/ESTADO-Y-RUMBO.md` + milestone Q1 | 3 | Cedente |
| 2+ ciclos EDD completos por maintainers | 3 | Maintainers |
| Release + fix end-to-end sin cedente | 4 | Maintainers |
| Retirada de accesos y anuncio | 5 | Ambos |

## Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| Bus factor se traslada (de idourra a 1 persona de BIREME) | Mínimo 2 maintainers desde Fase 1; CODEOWNERS por equipo, no por persona |
| La disciplina EDD se diluye sin el cedente | Branch protection + checks obligatorios (la regla la aplica GitHub); plantilla de PR con checklist; test de contrato de eval-specs en CI |
| Conocimiento tácito no transferido | Instalación limpia por BIREME en Fase 0; simulacro de incidencia en Fase 4; grabaciones |
| Ruptura silenciosa de `report.json` para consumidores (LILACS/BIREME) | Contrato documentado + política de versionado + pregunta explícita en plantilla de PR |
| Dependencia de credenciales personales | Auditoría y regeneración de todos los secrets en Fase 1 |

---

*Documento vivo: se actualiza por PR como cualquier otro cambio del repo.*
