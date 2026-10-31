Sistema Inteligente de Información Ambiental Municipal (Semillero IA)Hackathon de Inteligencia Artificial para el Bien Común · Universidad IberoamericanaPlataforma de gobernanza de datos territoriales y consulta ambiental multiagente. Integra y homologa información oficial de INEGI, CONAGUA y SEMARNAT a escala municipal, garantizando trazabilidad, cero alucinaciones y proyecciones paramétricas transparentes en tiempo real.🏛️ 1. Descripción del ProyectoEl análisis del estrés ambiental y la planeación hídrica municipal en México enfrenta tres grandes barreras:Dispersión y heterogeneidad de fuentes: INEGI opera por divisiones político-administrativas (CVEGEO), mientras CONAGUA gestiona el recurso por cuencas y acuíferos supra-municipales.Riesgo crítico de alucinación: Las aplicaciones estándar basadas en LLMs tienden a inferir o inventar datos poblacionales y balances hídricos ante la ausencia de registros directos.Falta de trazabilidad: Dificultad para auditar de qué columna, año y base institucional se extrajo cada métrica.Esta solución implementa una arquitectura multiagente orquestada en Saptiva conectada a un frontend analítico con simulador paramétrico, resolviendo de raíz estas deficiencias.🧠 2. Arquitectura del Sistema MultiagenteEl flujo de procesamiento opera de manera desacoplada y auditable:flowchart TD
    Usuario([Usuario / Frontend Web]) -->|Consulta en lenguaje natural| Router[Agente Router / Enrutador]
    
    subgraph Saptiva Engine - Orquestación Multiagente
        Router -->|Extracción de Entidad y Municipio| Catalogo[Agente de Catálogo Geoestadístico]
        Catalogo -->|Resolución estricta CVEGEO| Lider[Agente Líder / Orquestador]
        
        Lider -->|Consulta Censal| APob[Agente Especialista: Demografía
iter_2020_municipal_basico.xlsx]
        Lider -->|Consulta Disponibilidad| AHid[Agente Especialista: Hídrico
panel_aquifer_municipality_year.csv]
        Lider -->|Consulta Tratamiento| ASan[Agente Especialista: Saneamiento
saneamiento_municipios_limpio.csv]
        Lider -->|Consulta Disposición| ARes[Agente Especialista: Residuos
residuos_municipios_limpio.csv]
        
        APob --> Consolidador[Consolidador de Auditoría y Metodología]
        AHid --> Consolidador
        ASan --> Consolidador
        ARes --> Consolidador
        
        Consolidador -->|Aplicación de Regla Hídrica Espacial| Salida[Reporte Final Estructurado]
    end
    
    Salida -->|Respuesta Síncrona Webhook HTTP| Frontend[Frontend Web / Simulador Paramétrico]
Funciones de los NodosAgente Router: Clasifica intenciones y neutraliza consultas ambiguas o fuera de dominio.Agente de Catálogo: Valida el nombre del municipio y le asigna su identificador geoestadístico oficial de 5 dígitos (CVEGEO), mitigando fallas tipográficas o de homónimos.Agentes Especialistas: Consultan únicamente los campos autorizados de sus respectivos archivos sin realizar inferencias desautorizadas.Consolidador / Orquestador: Ensambla la respuesta, documenta la ficha de auditoría (archivo, columna y operación) y aplica la Regla Hídrica Espacial.💧 3. Metodología de Gobernanza y AuditoríaA. Regla Hídrica Espacial (Acuíferos vs. Municipios)Los límites de los acuíferos subterráneos no respetan fronteras municipales. Ante consultas de disponibilidad media anual subterránea ($DMA_{neta}$), el sistema no comete el error de truncar el dato:Reporta el balance hídrico oficial total del acuífero determinado por CONAGUA.Cruza la capa crosswalk_aquifer_municipality.csv para declarar el porcentaje de sobreposición espacial municipal (overlap_pct_aquifer).Evita divisiones arbitrarias y deja constancia metodológica de la escala hidrogeológica.B. Proyección Paramétrica TransparentePara cumplir con las normas de auditoría, los agentes de Saptiva declaran formalmente Dato no disponible en la fuente ante años no registrados en el Censo INEGI 2020. En paralelo, la interfaz web ejecuta una estimación paramétrica complementaria:Crecimiento Poblacional Geométrico:
$$P_t = P_{2020} \times (1 + r)^{(t - 2020)}$$
(Tasa $r$ ajustable por el usuario).Presión Hídrica Proyectada: Evaluación de estrés sobre la línea base $DMA_{neta}$ CONAGUA 2023.Generación de Residuos Sólidos Urbanos (RSU): Estimación con base en el factor oficial SEMARNAT ($0.944\text{ kg/hab/día}$).📂 4. Estructura de CarpetasC:\Escuela\Curso IA\Hackathon\
├── README.md                                # Documentación formal del proyecto
├── index.html                               # Aplicación web + Simulador Paramétrico
├── data/                                    # Datasets fuente normalizados
│   ├── crosswalk_aquifer_municipality.csv   # Cruces geoespaciales acuífero-municipio
│   ├── panel_aquifer_municipality_year.csv  # Series de disponibilidad CONAGUA
│   ├── saneamiento_municipios_limpio.csv    # Plantas de tratamiento en operación
│   ├── residuos_municipios_limpio.csv       # Toneladas de residuos recolectadas
│   └── iter_2020_municipal_basico.xlsx     # Censo de Población y Vivienda 2020 (INEGI)
└── docs/                                    # Diagramas y evidencias de flujo
🚀 5. Instrucciones de Ejecución LocalAbrir una terminal en el directorio del proyecto:cd "C:\Escuela\Curso IA\Hackathon"
Iniciar un servidor HTTP local ligero:# Opción con Python:
python -m http.server 8000

# O bien con Node.js:
npx serve .
Acceder en el navegador web a:http://localhost:8000
Ingresar cualquier municipio de prueba (ej. Hermosillo, Asientos, Zacatlán, Metepec) para recibir la ficha técnica auditada y el panel interactivo de proyección.