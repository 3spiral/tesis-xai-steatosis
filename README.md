# Sistema de IA Explicable para Esteatosis Hepática en Ecografía

Repositorio de desarrollo asociado al proyecto de tesis:

**“Diseño y Validación de un Sistema de Soporte a la Decisión Clínica Basado en IA Explicable para el Diagnóstico Ecográfico de Esteatosis Hepática”.**

El proyecto estudia el uso de modelos de Deep Learning sobre ecografías hepáticas en modo B, junto con técnicas de explicabilidad como Grad-CAM, para apoyar la identificación de esteatosis hepática.

---

## Estado del proyecto

Actualmente se encuentra implementada la etapa de preparación y control de calidad del dataset BEHSOF.

El pipeline disponible incluye:

- inventario automático del conjunto de datos;
- auditoría técnica de imágenes;
- revisión manual de casos seleccionados;
- exclusión documentada de imágenes técnicamente incompatibles;
- generación de un manifest maestro;
- definición del target binario;
- partición reproducible mediante validación cruzada estratificada a nivel de paciente.

El entrenamiento y evaluación de los modelos todavía se encuentra en desarrollo.

---

## Datasets

### BEHSOF

BEHSOF se utiliza como dataset principal de desarrollo.

El problema se formula como clasificación binaria a partir de la variable `Fibroscan S`:

- `0`: S0 / S1
- `1`: S2 / S3

Los pacientes del centro BEH se utilizan para desarrollo y validación cruzada.

Los pacientes del centro TAL se reservan como conjunto de evaluación externa.

### Byra

El dataset de Byra et al. se reserva para una etapa posterior de evaluación externa independiente.

Los datasets originales **no se incluyen en este repositorio**.

Deben obtenerse desde sus respectivas fuentes y almacenarse localmente.

---

## Estructura del proyecto

```text
tesis-xai-steatosis/
│
├── src/
│   ├── settings.py
│   └── data/
│       ├── inventory_behsof.py
│       ├── audit_behsof_quality.py
│       ├── make_tal_contact_sheets.py
│       ├── make_beh_sample_contact_sheets.py
│       ├── make_beh_outlier_sheet.py
│       └── build_behsof_manifest.py
│
├── metadata/
│   ├── BEHSOF_manifest.csv
│   ├── BEH_patient_folds.csv
│   └── qc/
│
├── artifacts/
│   └── qc/
│
├── docs/
├── results/
│
├── config.example.yaml
├── requirements.txt
├── .gitignore
└── README.md
```

Los datos médicos originales se mantienen fuera del repositorio:

```text
tesis-xai-steatosis-data/
│
├── raw/
│   ├── BEHSOF/
│   └── Byra/
│
├── interim/
├── processed/
└── models/
```

---

## Configuración

El proyecto utiliza dos archivos de configuración:

```text
config.example.yaml
config.local.yaml
```

`config.example.yaml` contiene una configuración reproducible de ejemplo y se encuentra versionado.

`config.local.yaml` contiene la configuración específica de la máquina local y está excluido mediante `.gitignore`.

La estructura recomendada es:

```text
project-root/
├── tesis-xai-steatosis/
└── tesis-xai-steatosis-data/
```

Con esta estructura puede utilizarse:

```yaml
paths:
  data_root: "../tesis-xai-steatosis-data"
```

---

## Instalación

Crear un entorno virtual:

```powershell
python -m venv .venv
```

Activarlo en Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Instalar dependencias:

```powershell
python -m pip install -r requirements.txt
```

Crear la configuración local:

```powershell
Copy-Item config.example.yaml config.local.yaml
```

Después, editar `config.local.yaml` si la ubicación local de los datasets es diferente.

---

## Pipeline de preparación de BEHSOF

### 1. Inventario

```powershell
python -m src.data.inventory_behsof
```

Genera:

```text
metadata/qc/BEHSOF_inventory.csv
```

### 2. Auditoría técnica automática

```powershell
python -m src.data.audit_behsof_quality
```

Genera:

```text
metadata/qc/BEHSOF_quality_audit.csv
```

La auditoría automática sirve para identificar candidatos que requieren inspección y no constituye una valoración clínica de las imágenes.

### 3. Revisión visual

Los scripts:

```text
make_tal_contact_sheets.py
make_beh_sample_contact_sheets.py
make_beh_outlier_sheet.py
```

fueron utilizados para generar artefactos destinados a la revisión visual técnica.

Los scripts incluyen protección contra sobrescritura para evitar reemplazar accidentalmente decisiones manuales de QC.

Los contact sheets no se versionan en Git debido a que contienen imágenes derivadas de los datasets médicos.

### 4. Manifest

```powershell
python -m src.data.build_behsof_manifest
```

Genera:

```text
metadata/BEHSOF_manifest.csv
metadata/BEH_patient_folds.csv
```

---

## Validación cruzada

La validación interna utiliza:

```text
5-fold StratifiedKFold
```

con:

```text
random_state = 42
```

La partición se realiza **a nivel de paciente**, no a nivel de imagen.

Esto evita que imágenes pertenecientes al mismo paciente aparezcan simultáneamente en entrenamiento y validación.

El centro TAL no participa en la selección de hiperparámetros ni en la validación cruzada.

---

## Estado actual del dataset BEHSOF

El inventario utilizado en el proyecto contiene:

- 113 pacientes;
- 1601 imágenes estáticas;
- 1388 imágenes BEH;
- 213 imágenes TAL.

Después del control técnico:

- 1566 imágenes incluidas;
- 35 imágenes excluidas;
- 112 pacientes elegibles.

Las exclusiones corresponden a:

- 26 composiciones dual-panel;
- 6 imágenes de baja información visual;
- 3 archivos no ecográficos.

---

## Reproducibilidad

El pipeline fue verificado después de migrar las rutas absolutas originales a la configuración centralizada actual.

Los archivos regenerados:

```text
BEHSOF_inventory.csv
BEHSOF_quality_audit.csv
BEHSOF_manifest.csv
BEH_patient_folds.csv
```

resultaron idénticos a sus versiones previamente validadas.

---

## Privacidad y datos

Los datasets médicos originales y los artefactos visuales derivados no se distribuyen mediante este repositorio.

El repositorio contiene únicamente código, configuración, metadatos de investigación y resultados necesarios para documentar y reproducir el pipeline.

---

## Autor

Sebastián Díaz  
Ingeniería de Software  
Universidad Nacional Mayor de San Marcos
