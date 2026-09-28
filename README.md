# TP1 — Baseline Clássico para Detecção de Fratura Cervical (RSNA 2022)

**Disciplina:** Tópicos Especiais em Sistemas de Informação — Prof. Décio Gonçalves de Aguiar Neto
**Curso:** Sistemas de Informação
**Grupo G5:** Pedro Victor da Silva Lima, Victor Pinheiro de Lima, Maria Letícia Pimentel Carlos
**Desafio:** [RSNA 2022 Cervical Spine Fracture Detection](https://www.kaggle.com/competitions/rsna-2022-cervical-spine-fracture-detection)

## Resumo do projeto

Este repositório contém o código de um baseline **clássico** (sem redes neurais profundas) para
detecção de fratura na coluna cervical em tomografias computadorizadas, usando engenharia de
características manuais (histograma de intensidade, LBP e HOG) combinada a classificadores
tradicionais de aprendizado de máquina (Regressão Logística, Random Forest e SVM).

## Origem dos dados

Os dados são do desafio [RSNA 2022 Cervical Spine Fracture Detection](https://www.kaggle.com/competitions/rsna-2022-cervical-spine-fracture-detection),
hospedado no Kaggle. É necessário:

1. Ter uma conta no Kaggle e aceitar as regras da competição na página acima;
2. Executar este notebook **dentro do ambiente Kaggle Notebooks**, com a competição
   `rsna-2022-cervical-spine-fracture-detection` adicionada como Input — os dados ficam
   disponíveis automaticamente em `/kaggle/input/competitions/rsna-2022-cervical-spine-fracture-detection/`,
   sem necessidade de download manual.

Não amostramos o dataset completo (dezenas de GB): selecionamos uma amostra estratificada de
**150 pacientes**, com **semente fixa (42)**, preservando a proporção de fratura do conjunto
original. O critério de amostragem e a partição treino/validação por paciente (nunca por corte,
para evitar vazamento de dados) são gerados automaticamente pelo próprio notebook, na primeira
célula de execução — não é necessário nenhum arquivo de amostra pré-gerado.

## Como reproduzir os resultados

1. Acesse o notebook público: **[link do notebook Kaggle — inserir aqui]**
2. Clique em "Copy & Edit" (ou "Fork") para criar sua própria cópia executável
3. Confirme que a competição `rsna-2022-cervical-spine-fracture-detection` está conectada em
   **Input → Competitions** (se não estiver, adicione pela busca)
4. Execute as células **em ordem, de cima para baixo** (não pule nenhuma). A célula de extração
   de características demora entre 10 e 30 minutos — isso é esperado
5. Ao final, os seguintes arquivos são gerados em `/kaggle/working/`:
   - `particao_congelada.csv` — amostra e partição treino/validação (semente 42)
   - `features_baseline.csv` — matriz de características (histograma + LBP + HOG) por paciente
   - `tabela_resultados.csv` — desempenho de cada modelo (AUC-ROC, F1, acurácia balanceada)
   - `ablacao.csv` — resultado do estudo de ablação por família de descritor
   - `curva_roc.png` — curva ROC comparando os modelos
   - `exemplo_preprocessamento.png`, `exemplos_por_classe.png`, `exemplo_erro.png` — figuras
     auxiliares para o artigo

Com a mesma semente (42), os números obtidos devem coincidir com os reportados no artigo,
dentro da variação esperada de bibliotecas/versões do ambiente Kaggle.

## Dependências

Todas as bibliotecas usadas já vêm pré-instaladas no ambiente padrão do Kaggle Notebooks:

```
pandas
numpy
scikit-learn
scikit-image
pydicom
matplotlib
```

Nenhuma instalação adicional é necessária (`pip install` não é usado neste projeto).

## Estrutura do pipeline

1. **Leitura de metadados** — `train.csv` da competição
2. **Amostragem estratificada e partição por paciente** — semente fixa, congelada antes de
   qualquer extração de características
3. **Pré-processamento** — leitura DICOM (`pydicom`), conversão para Unidades Hounsfield (HU)
   via `RescaleSlope`/`RescaleIntercept`, janelamento ósseo (centro 400, largura 1800),
   redimensionamento para 128×128
4. **Agregação 2.5D** — até 15 cortes por paciente, igualmente espaçados; características
   extraídas por corte e agregadas por média/desvio-padrão
5. **Extração de características** — histograma de intensidade, LBP (textura), HOG (forma)
6. **Modelagem** — baseline trivial (classe majoritária), Regressão Logística, Random Forest,
   SVM (RBF), todos com `class_weight="balanced"`
7. **Validação** — validação cruzada estratificada (5 dobras) no treino + ajuste de
   hiperparâmetros via `GridSearchCV` no Random Forest + avaliação final única na partição de
   validação (nunca usada em ajuste)
8. **Ablação e análise de erro** — comparação por família de descritor e inspeção qualitativa
   de casos classificados incorretamente

## Uso de IA generativa

Ferramentas de IA generativa foram utilizadas como apoio na elaboração do código de
extração de características, treinamento de modelos e revisão de texto. Todo código foi
executado e os resultados foram verificados pelos autores.

## Licença dos dados

Os dados pertencem à RSNA e à American Society of Neuroradiology (ASNR), distribuídos sob os
termos de uso da competição no Kaggle. Este repositório não redistribui nenhum dado bruto —
apenas o código necessário para reproduzir os experimentos a partir da fonte original.
