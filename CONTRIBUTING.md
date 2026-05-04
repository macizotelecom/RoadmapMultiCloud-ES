# 🤝 Guía de Contribución — RoadmapMultiCloud-ES

Gracias por querer contribuir al proyecto. Esta guía explica cómo hacerlo
de forma que tu aportación sea útil, coherente y fácil de integrar.

---

## 📋 Antes de empezar

1. Lee el [README principal](README.md) para entender la estructura del proyecto
2. Revisa si ya existe un issue abierto sobre lo que quieres aportar
3. Si es una contribución grande, abre un issue primero para discutirlo

---

## 🏗️ Tipos de contribuciones bienvenidas

### ✅ Muy valoradas
- **Capturas actualizadas** — los portales cloud cambian frecuentemente
- **Correcciones de costos** — los precios cloud se actualizan constantemente
- **Labs equivalentes** en AWS, GCP u OCI para módulos que solo tienen Azure
- **Errores reales documentados** — troubleshooting basado en experiencia propia
- **Mejoras de IaC** — scripts Bicep, Terraform o CloudFormation más robustos
- **Correcciones de errores** en comandos CLI o pasos desactualizados

### ⚠️ Requieren discusión previa (abre un issue)
- Añadir una nueva fase o certificación no contemplada en el roadmap
- Cambiar la estructura de carpetas
- Modificar la plantilla universal de lab
- Añadir un nuevo cloud no contemplado

### ❌ No se aceptan
- Contenido en inglés (el proyecto es íntegramente en español)
- Labs sin capturas propias (no se aceptan capturas de otros repositorios)
- Contenido con fines comerciales o promocionales
- Modificaciones que rompan la coherencia con el patrón base→avanzado→FinOps

---

## 🔄 Proceso para contribuir

### 1. Fork y clone
```bash
# Haz fork del repositorio desde GitHub
git clone https://github.com/TU-USUARIO/RoadmapMultiCloud-ES.git
cd RoadmapMultiCloud-ES
```

### 2. Crea una rama descriptiva
```bash
# Sigue este patrón de nombres:
git checkout -b fix/az104-lab07-capturas-actualizadas
git checkout -b add/aws-saa-lab04-vpc-equivalente
git checkout -b update/finops-storage-precios-2025
```

### 3. Aplica la plantilla de lab
Usa siempre [TEMPLATE-LAB.md](TEMPLATE-LAB.md) como base para cualquier lab nuevo.
No inventes estructuras propias — la coherencia entre labs es lo que hace útil el proyecto.

### 4. Requisitos de calidad

Antes de hacer commit, verifica:

- [ ] El contenido está completamente en español
- [ ] Incluye capturas propias (no copiadas de otros repos)
- [ ] El costo real está documentado (aunque sea $0.01)
- [ ] Los comandos CLI están probados y funcionan
- [ ] Sigue el patrón: lab-base / lab-avanzado / finops
- [ ] El encabezado del lab indica: Fase / Cert / Lab número / Tipo

### 5. Commit con mensaje descriptivo
```bash
# Sigue este formato:
git commit -m "feat(az104): añade lab-07 avanzado con escenario logística"
git commit -m "fix(az104): actualiza capturas lab-04 portal actualizado 2025"
git commit -m "docs(finops): añade comparativa storage Azure vs AWS vs GCP"
git commit -m "add(aws-saa): añade lab equivalente VPC para lab-04 AZ-104"
```

### 6. Pull Request
- Título claro: `[F2-AZ104] Lab 07 avanzado — escenario empresa logística`
- Descripción: qué cambiaste, por qué, cómo lo probaste
- Si actualizas costos, incluye la fecha y región donde los verificaste

---

## 📐 Estándares de formato

### Encabezado obligatorio en cada lab
```markdown
---
Fase: F2 — Administración
Certificación: AZ-104 / SAA-C03 / ACE / OCI-Admin
Lab: 07
Tipo: base | avanzado | finops
Última actualización: YYYY-MM
Costo real documentado: $X.XX USD
Región usada: East US / eu-west-1 / europe-west1 / etc.
---
```

### Idioma y tono
- Español neutro (no regionalismos que excluyan a lectores de otros países)
- Imperativo para instrucciones: "Haz clic en...", "Ejecuta el comando..."
- Segunda persona singular: "tú" o directamente imperativo, nunca "usted"

### Capturas de pantalla
- Formato: PNG, máximo 1920×1080
- Nombre de archivo: `paso-01-crear-storage-account.png`
- Carpeta: `lab-base/capturas/` o `lab-avanzado/capturas/`
- Obligatorio: redactar datos personales o sensibles antes de subir

### Costos reales
- Siempre con captura del portal de billing
- Indicar región y fecha de ejecución
- Incluir el tiempo que tardó el lab para referencia

---

## 🐛 Reportar problemas

Usa los issues de GitHub con estas etiquetas:

| Etiqueta | Cuándo usarla |
|----------|---------------|
| `bug` | Comando que no funciona, paso incorrecto |
| `outdated` | Captura o precio desactualizado |
| `enhancement` | Mejora de contenido existente |
| `new-lab` | Propuesta de lab nuevo |
| `finops` | Relacionado con análisis de costos |
| `question` | Duda sobre el contenido de un lab |

---

## 📄 Licencia de contribuciones

Al contribuir, aceptas que tu aportación se publica bajo la misma
licencia del proyecto: **CC BY-NC-SA 4.0**.

Si incluyes código (scripts, IaC), ese código específico se publica
bajo **MIT License** para facilitar su reutilización técnica.

---

*¿Dudas? Abre un issue con la etiqueta `question`.*
