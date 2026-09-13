# Atividade 1 — Fundamentos de Processamento de Imagens

**Curso:** Visão Computacional — Residência em TIC43
**Professor:** Alyson Bezerra
**Unidade 1 | Capítulo 1 | Tarefa 2**

**Notebook (Google Colab):** https://colab.research.google.com/drive/1l6Aga7io5GFzkMwaYwZ984XC1xOctk0a?usp=sharing

## Objetivo

Construir um mini dataset próprio, com 2 classes e 10 imagens no total, aplicando na prática os fundamentos de Processamento Digital de Imagens vistos em aula:

- Resolução e profundidade de cor
- Conversões de espaço de cor (RGB, HSV, escala de cinza)
- Quantização de níveis de cinza
- Formatos de arquivo (JPEG x PNG)
- Aquisição controlada de imagens (iluminação, distância e ângulo)
- Estruturação do dataset em padrão de Machine Learning

## Stack utilizada

[![Python](https://img.shields.io/badge/Python-3-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Google Colab](https://img.shields.io/badge/Google_Colab-Notebook-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![Google Drive](https://img.shields.io/badge/Google_Drive-Storage-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)](https://www.google.com/drive/)
[![OpenCV](https://img.shields.io/badge/OpenCV-cv2-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)](https://opencv.org/)
[![NumPy](https://img.shields.io/badge/NumPy-Array_ops-013243?style=for-the-badge&logo=numpy&logoColor=white)](https://numpy.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualização-11557C?style=for-the-badge&logo=plotly&logoColor=white)](https://matplotlib.org/)
[![Pillow](https://img.shields.io/badge/Pillow-pillow--heif-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python-pillow.org/)

## Classes definidas

| Classe | Objeto | Imagens |
|---|---|---|
| Classe A | Controle | `img001` a `img005` |
| Classe B | Nintendo Switch | `img006` a `img010` |

## Aquisição das imagens

- **Dispositivo:** Samsung A34 5G (câmera nativa, sem filtros automáticos, embelezamento desativado)
- Capturas variando cenário de iluminação (luz natural, sombra, luz artificial e mista) e distância/ângulo, seguindo o mesmo padrão para as duas classes

## Preparação do ambiente

Antes das transformações, o notebook monta o Google Drive (`drive.mount`) para acessar o dataset em `/content/drive/MyDrive/dataset` e trata imagens capturadas em formato HEIC (padrão de câmeras Apple/Android mais recentes), que o OpenCV não lê nativamente:

- `pillow_heif.register_heif_opener()` — registra o suporte a HEIC no Pillow
- `PIL.Image.open(...).convert('RGB')` — abre o arquivo HEIC e converte para RGB
- `Image.save(..., 'JPEG', quality=95)` — salva a versão convertida em JPEG
- Função própria `converter_pasta(pasta)` — percorre cada pasta de classe e converte todos os `.heic` encontrados para `.jpg`, removendo o original com `os.remove`

## Transformações aplicadas

**A. Análise de resolução — `analisar_resolucao()`**
Lê a imagem original com `cv2.imread` e converte de BGR para RGB com `cv2.cvtColor(img, cv2.COLOR_BGR2RGB)` (o OpenCV carrega no formato BGR por padrão, mas o Matplotlib espera RGB). Em seguida gera duas versões reduzidas com `cv2.resize`, usando metade (`largura//2, altura//2`) e um quinto (`largura//5, altura//5`) das dimensões originais, e exibe as três versões lado a lado com `matplotlib.pyplot.subplots`. A interpretação registrada discute a perda progressiva de detalhes finos e o possível impacto em modelos de Visão Computacional.

**B. Espaço de cor — `analisar_espaco_cor()`**
A partir da imagem em BGR, gera três representações com `cv2.cvtColor`:
- `cv2.COLOR_BGR2RGB` — espaço RGB (cores naturais)
- `cv2.COLOR_BGR2HSV` — separa matiz (H), saturação (S) e valor/brilho (V)
- `cv2.COLOR_BGR2GRAY` — escala de cinza (apenas intensidade luminosa)

As três versões são exibidas com `plt.imshow`, usando `cmap='gray'` na versão em escala de cinza. O notebook discute quando cada espaço é mais adequado (ex.: HSV para segmentação por cor, escala de cinza para reduzir dimensionalidade).

**C. Quantização — `analisar_quantizacao()`**
Carrega a imagem já em escala de cinza (`cv2.imread(caminho_img, 0)`) e reduz os níveis de intensidade por meio de operações aritméticas com NumPy:
- 64 níveis: `(img // 4) * 4`
- 32 níveis: `(img // 8) * 8`
- 2 níveis (binarização): `((img > 127) * 255).astype(np.uint8)`

As quatro versões (256, 64, 32 e 2 níveis) são comparadas visualmente, evidenciando o efeito de posterização conforme os níveis diminuem.

**D. Formato de arquivo — `analisar_formatos()`**
Salva a mesma imagem em dois formatos com `cv2.imwrite`:
- JPEG com compressão, usando o parâmetro `cv2.IMWRITE_JPEG_QUALITY` definido como 30
- PNG sem perdas (parâmetros padrão)

O tamanho de cada arquivo é obtido com `os.path.getsize` (convertido para KB) e comparado entre os dois formatos, servindo de base para a hipótese sobre impacto de armazenamento em pipelines de PDI.

## Organização final do dataset

Duas funções fecham o notebook, responsáveis por padronizar e revisar visualmente o dataset final:

- `padronizar_nomes(pasta, prefixo_inicio)` — lista os arquivos `.jpg` de cada classe com `os.listdir` e renomeia sequencialmente para o padrão `imgXXX_original.jpg` com `os.rename`, garantindo a numeração `img001` a `img010`
- `exibir_dataset(pasta_A, pasta_B)` — monta uma grade 2x5 com `plt.subplots(2, 5, ...)`, exibindo todas as imagens das duas classes lado a lado (após conversão BGR → RGB) como conferência final do mini dataset

## Estrutura do mini dataset

```
dataset/
├── classe_A/
│   ├── img001_original.jpg
│   ├── img002_original.jpg
│   ├── img003_original.jpg
│   ├── img004_original.jpg
│   └── img005_original.jpg
└── classe_B/
    ├── img006_original.jpg
    ├── img007_original.jpg
    ├── img008_original.jpg
    ├── img009_original.jpg
    └── img010_original.jpg
```

## Conclusões gerais

O notebook segue a estrutura Problema → Código → Visualização → Interpretação em cada etapa, relacionando as decisões técnicas de resolução, espaço de cor, quantização e formato de arquivo com seus impactos práticos no desempenho de sistemas de Visão Computacional.