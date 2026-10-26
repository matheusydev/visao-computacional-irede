# Visão Computacional — Residência em TIC43

Repositório com as atividades práticas da disciplina de **Visão Computacional** da Residência em TIC43. Cada atividade tem uma pasta própria com o notebook (`.ipynb`) e um README que documenta o objetivo, as técnicas aplicadas, os resultados e as conclusões.

Grande parte das atividades usa um **mini dataset próprio**, construído na Atividade 1, que é reutilizado nas seguintes. Assim, as mesmas imagens passam por todo o caminho da disciplina, dos fundamentos de processamento de imagens até a extração e comparação de descritores.

## Stack utilizada

[![Python](https://img.shields.io/badge/Python-3-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Google Colab](https://img.shields.io/badge/Google_Colab-Notebook-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![Google Drive](https://img.shields.io/badge/Google_Drive-Storage-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)](https://www.google.com/drive/)
[![OpenCV](https://img.shields.io/badge/OpenCV-cv2-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)](https://opencv.org/)
[![scikit-image](https://img.shields.io/badge/scikit--image-Segmentação_e_descritores-FF6F00?style=for-the-badge)](https://scikit-image.org/)
[![SciPy](https://img.shields.io/badge/SciPy-Morfologia-8CAAE6?style=for-the-badge&logo=scipy&logoColor=white)](https://scipy.org/)
[![NumPy](https://img.shields.io/badge/NumPy-Arrays_e_FFT-013243?style=for-the-badge&logo=numpy&logoColor=white)](https://numpy.org/)
[![pandas](https://img.shields.io/badge/pandas-Tabelas-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualização-11557C?style=for-the-badge&logo=plotly&logoColor=white)](https://matplotlib.org/)
[![Pillow](https://img.shields.io/badge/Pillow-HEIC_e_metadados-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python-pillow.org/)

## Atividades

| # | Atividade | Unidade / Capítulo | Professor | Pasta | Notebook |
|---|---|---|---|---|---|
| 1 | Fundamentos de Processamento de Imagens | Unidade 1 · Capítulo 1 | Alyson Bezerra | [Abrir](<Atividade 1 - Fundamentos de Processamento de Imagens/>) | [Colab](https://colab.research.google.com/drive/1l6Aga7io5GFzkMwaYwZ984XC1xOctk0a?usp=sharing) |
| 2 | Suavização, Remoção de Ruído e Detecção de Bordas | Unidade 2 · Capítulo 1 | Rafael Carmo | [Abrir](<Atividade 2 - Suavização, Remoção de Ruído e Detecção de Bordas/>) | [Colab](https://colab.research.google.com/drive/1Mmdb5uxCBdv4UcXfAt-LxP4AHcdnA_nx?usp=sharing) |
| 3 | Filtragem de Imagens e Análise de Ruído | Unidade 2 · Capítulo 1 | Rafael Carmo | [Abrir](<Atividade 3 - Filtragem de Imagens e Análise de Ruído/>) | [Colab](https://colab.research.google.com/drive/1B_7wCsD3so0247m_bhUhoRaMhIZv82AT?usp=sharing) |
| 4 | Segmentação, Operações Morfológicas e Extração de Características | Unidade 3 · Capítulo 1 | Rafael Carmo | [Abrir](<Atividade 4 - Segmentação, Operações Morfológicas e Extração de Características/>) | [Colab](https://colab.research.google.com/drive/1dlkoc59FaXFjHOKJxhc6Tr3up7XexxTf?usp=sharing) |
| 5 | Análise Global, Local e de Textura em Mini Dataset Próprio | Unidade 3 · Capítulo 2 | Rafael Carmo | [Abrir](<Atividade 5 - Análise Global, Local e de Textura em Mini Dataset Próprio/>) | [Colab](https://colab.research.google.com/drive/1h8KOpkSLbrCYsxeNeO6gZnvtJze2IpoU?usp=sharing) |

### Resumo de cada atividade

**Atividade 1 — Fundamentos de Processamento de Imagens**
Construção do mini dataset (2 classes, 10 imagens) com aquisição controlada, e análise prática de resolução, espaços de cor (RGB, HSV e escala de cinza), quantização de níveis de cinza e formatos de arquivo (JPEG × PNG).

**Atividade 2 — Suavização, Remoção de Ruído e Detecção de Bordas**
Filtros de suavização (média, Gaussiano e mediana) e operadores de borda (Sobel, Prewitt e Canny) aplicados às imagens de menor iluminação do dataset, com análise quantitativa e avaliação do efeito da suavização prévia sobre a detecção de bordas.

**Atividade 3 — Filtragem de Imagens e Análise de Ruído**
Inserção de ruído gaussiano e sal e pimenta e filtragem **no domínio da frequência** (passa-baixa gaussiano 3×3 e 7×7 e passa-alta Laplaciano via FFT), medindo o impacto de cada etapa em algoritmos clássicos (Canny e Otsu). Os detalhes de baixo contraste do controle desaparecem sob o ruído antes das estampas de alto contraste do Switch.

**Atividade 4 — Segmentação, Operações Morfológicas e Extração de Características**
Pipeline completo sobre a imagem `coins` do scikit-image: segmentação por Otsu, limiarização adaptativa e watershed, refinamento morfológico, componentes conectados e extração de nove características por objeto. O watershed com marcadores foi o único método confiável diante da iluminação não uniforme.

**Atividade 5 — Análise Global, Local e de Textura em Mini Dataset Próprio**
Comparação crítica entre descritores globais (estatísticas e histogramas), locais (Shi-Tomasi, LoG, ORB + matching) e de textura (GLCM e LBP) no dataset próprio. Os descritores globais e de textura reconheceram o **cenário**; só o ORB com verificação geométrica reconheceu o **objeto**, sem nenhuma classificação errada.

## Mini dataset próprio

Construído na Atividade 1 e reutilizado nas Atividades 2, 3 e 5.

| Classe | Objeto | Imagens |
|---|---|---|
| Classe A | Controle | `img001` a `img005` |
| Classe B | Nintendo Switch | `img006` a `img010` |

- **Dispositivo:** Samsung Galaxy A34 5G (fotos de 48 MP, convertidas de HEIC para JPEG)
- **Aquisição:** cinco cenários de captura (luz natural, sombra, luz artificial noturna, luz artificial e luz mista), **repetidos nas duas classes**. Cada imagem da Classe A tem uma correspondente da Classe B tirada no mesmo cenário (img001 ↔ img006, img002 ↔ img007, …), o que permite separar o efeito do objeto do efeito da cena.

## Estrutura do repositório

```
visao-computacional-irede/
├── README.md
├── Atividade 1 - Fundamentos de Processamento de Imagens/
│   ├── Atividade_1_Visão_Computacional.ipynb
│   ├── README.md
│   └── dataset/
│       ├── classe_A/   img001_original.jpg … img005_original.jpg
│       └── classe_B/   img006_original.jpg … img010_original.jpg
├── Atividade 2 - Suavização, Remoção de Ruído e Detecção de Bordas/
│   ├── Atividade_2_Visao_Computacional.ipynb
│   └── README.md
├── Atividade 3 - Filtragem de Imagens e Análise de Ruído/
│   ├── Atividade_3_Visao_Computacional.ipynb
│   └── README.md
├── Atividade 4 - Segmentação, Operações Morfológicas e Extração de Características/
│   ├── Atividade_4_Visao_Computacional.ipynb
│   └── README.md
└── Atividade 5 - Análise Global, Local e de Textura em Mini Dataset Próprio/
    ├── Atividade_5_Visao_Computacional.ipynb
    └── README.md
```

## Como executar

1. Abra o notebook desejado pelo link do Colab na tabela de atividades (ou faça upload do `.ipynb` no [Google Colab](https://colab.research.google.com/)).
2. Para as atividades que usam o mini dataset (1, 2, 3 e 5), copie a pasta `dataset/` da Atividade 1 para a raiz do seu Google Drive, de forma que o caminho fique `MyDrive/dataset/classe_A` e `MyDrive/dataset/classe_B`.
3. Execute as células em ordem e autorize o acesso ao Google Drive quando solicitado.

A Atividade 4 usa uma imagem padrão do scikit-image (`skimage.data.coins()`) e não depende de arquivos externos.

## Autor

**Matheus Ylan Araújo Moraes** — Graduando em Análise e Desenvolvimento de Sistemas (ADS) pelo Instituto Federal do Piauí (IFPI).

[![GitHub](https://img.shields.io/badge/GitHub-matheusydev-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/matheusydev)
