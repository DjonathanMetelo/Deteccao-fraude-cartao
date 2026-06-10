# Detecção de Fraudes em Transações Bancárias

Projeto final de Ciência de Dados voltado para a construção de um modelo preditivo capaz de identificar fraudes em transações de cartão de crédito.

## Contexto do projeto

Este projeto simula uma demanda real de negócio apresentada por um stakeholder da área de riscos de um banco fictício, o **Banco Global Trust**.

O banco processou **284.807 transações de cartão de crédito** durante dois dias de setembro de 2023. Desse total, apenas **492 transações foram identificadas como fraude**, representando aproximadamente **0,17%** da base.

O principal desafio do projeto é lidar com uma base extremamente desbalanceada, na qual as fraudes são raras, mas representam alto risco financeiro e operacional para o banco.

## Problema de negócio

Fraudes em transações bancárias estão se tornando cada vez mais sofisticadas e frequentes. Para o banco, detectar essas fraudes rapidamente é essencial para:

* proteger os clientes;
* reduzir prejuízos financeiros;
* preservar a integridade das operações;
* garantir uma experiência segura nas transações.

A principal preocupação do stakeholder é evitar que fraudes reais passem despercebidas. Por isso, o projeto busca minimizar os **falsos negativos**, ou seja, casos em que uma transação fraudulenta é classificada como legítima.

Ao mesmo tempo, também é necessário controlar os **falsos positivos**, pois classificar muitas transações legítimas como fraude pode prejudicar a experiência do cliente e gerar custos operacionais.

## Objetivo

Desenvolver um modelo de Machine Learning capaz de classificar transações como fraudulentas ou legítimas, priorizando a identificação correta das fraudes.

O modelo deve equilibrar:

* alto recall para detectar o maior número possível de fraudes;
* boa precision para evitar excesso de alertas falsos;
* bom F1-score para equilibrar recall e precision;
* baixo número de falsos negativos;
* controle dos falsos positivos.

## Sobre a base de dados

A base contém transações de cartão de crédito com as seguintes características:

* `Time`: tempo em segundos desde a primeira transação;
* `Amount`: valor da transação;
* `Class`: variável alvo, sendo:

  * `0`: transação legítima;
  * `1`: transação fraudulenta;
* `V1` a `V28`: variáveis numéricas transformadas por PCA.

As variáveis `V1` a `V28` já passaram por uma transformação PCA, portanto não representam variáveis originais diretamente interpretáveis. Isso limita parte da análise exploratória, mas mantém a utilidade da base para modelagem preditiva.

## Etapas realizadas

O projeto foi desenvolvido de forma autônoma, seguindo as principais etapas de um projeto de Ciência de Dados:

1. Entendimento do problema de negócio;
2. Carregamento e inspeção da base;
3. Análise do desbalanceamento da variável alvo;
4. Análise exploratória dos dados;
5. Tratamento das variáveis `Time` e `Amount`;
6. Criação das variáveis `Time_hours` e `Amount_log`;
7. Separação entre variáveis explicativas e variável alvo;
8. Divisão entre treino e teste com estratificação;
9. Criação de modelo baseline;
10. Teste de diferentes algoritmos;
11. Aplicação de técnicas para lidar com desbalanceamento;
12. Validação cruzada;
13. Ajuste de hiperparâmetros;
14. Escolha do modelo final;
15. Avaliação dos resultados e considerações finais.

## Modelos avaliados

Foram testados diferentes modelos de classificação:

* Regressão Logística;
* Regressão Logística com balanceamento;
* Random Forest;
* Random Forest com balanceamento;
* XGBoost;
* XGBoost com balanceamento;
* SVM Linear;
* SVM Linear com balanceamento;
* Random Forest com SMOTE;
* Random Forest com SMOTE e balanceamento;
* XGBoost com SMOTE;
* XGBoost com SMOTE e balanceamento.

A Regressão Logística foi utilizada como modelo baseline, servindo como referência inicial para comparação com modelos mais robustos.

## Tratamento do desbalanceamento

Como a base possui apenas cerca de 0,17% de fraudes, foram avaliadas diferentes estratégias para lidar com o desbalanceamento:

* `class_weight='balanced'`;
* `scale_pos_weight` no XGBoost;
* SMOTE para criação de amostras sintéticas da classe minoritária.

O SMOTE foi aplicado apenas ao conjunto de treino, mantendo o conjunto de teste sem reamostragem, para garantir uma avaliação mais realista do modelo.

## Validação cruzada

Após os testes iniciais, os modelos mais promissores foram comparados com validação cruzada. O objetivo foi verificar se os resultados se mantinham consistentes em diferentes divisões dos dados.

O duelo principal foi entre:

* Random Forest com SMOTE;
* XGBoost com SMOTE e balanceamento.

O XGBoost SMOTE Balanced apresentou maior recall, mas gerou muitos falsos positivos. O Random Forest SMOTE apresentou melhor equilíbrio geral entre recall, precision, F1-score, falsos negativos e falsos positivos.

## Ajuste de hiperparâmetros

O modelo Random Forest com SMOTE foi selecionado para ajuste de hiperparâmetros com `RandomizedSearchCV`.

Foram testadas diferentes combinações de parâmetros, como:

* `n_estimators`;
* `max_depth`;
* `min_samples_split`;
* `min_samples_leaf`;
* `max_features`;
* `criterion`;
* `bootstrap`;
* `max_samples`;
* `ccp_alpha`.

Após os ajustes, a melhor versão encontrada foi o **Random Forest SMOTE v3**.

## Modelo final

O modelo final escolhido foi:

**Random Forest com SMOTE - v3**

Hiperparâmetros selecionados:

```python
{
    'n_estimators': 250,
    'min_samples_split': 5,
    'min_samples_leaf': 1,
    'max_samples': None,
    'max_features': 'sqrt',
    'max_depth': 25,
    'criterion': 'entropy',
    'ccp_alpha': 0.0,
    'bootstrap': True
}
```

## Resultados do modelo final

Resultado no conjunto de teste:

| Métrica   |    Valor |
| --------- | -------: |
| Accuracy  | 0.999631 |
| Recall    | 0.887755 |
| Precision | 0.896907 |
| F1-score  | 0.892308 |

Matriz de confusão:

| Resultado             | Quantidade |
| --------------------- | ---------: |
| Verdadeiros negativos |     56.854 |
| Falsos positivos      |         10 |
| Falsos negativos      |         11 |
| Verdadeiros positivos |         87 |

O modelo conseguiu identificar **87 das 98 fraudes** presentes no conjunto de teste, deixando passar **11 fraudes** e gerando apenas **10 falsos positivos**.

## Conclusão

O Random Forest SMOTE v3 foi escolhido como modelo final por apresentar o melhor equilíbrio entre as necessidades do negócio e as métricas de avaliação.

Embora o objetivo principal fosse minimizar falsos negativos, também era importante evitar um número excessivo de falsos positivos. O modelo final conseguiu detectar a maior parte das fraudes sem gerar muitos alertas incorretos, mostrando-se uma solução adequada para apoiar a área de riscos do banco.

A acurácia não foi utilizada como principal métrica de decisão, pois a base é altamente desbalanceada. Em vez disso, a escolha do modelo considerou principalmente recall, precision, F1-score, falsos negativos e falsos positivos.

## Possíveis melhorias futuras

Em um cenário real de produção, algumas melhorias poderiam ser realizadas:

* testar diferentes thresholds de decisão;
* avaliar o custo financeiro de falsos positivos e falsos negativos;
* monitorar o desempenho do modelo ao longo do tempo;
* reavaliar o modelo com dados mais recentes;
* integrar o modelo a uma esteira de análise de risco;
* criar dashboards para acompanhamento das transações suspeitas;
* validar as previsões com especialistas da área de fraude.

## Tecnologias utilizadas

* Python
* Pandas
* NumPy
* Matplotlib
* Seaborn
* Scikit-learn
* Imbalanced-learn
* XGBoost

## Observação

Este projeto foi desenvolvido como uma simulação de uma demanda real de negócio. O modelo deve ser interpretado como uma ferramenta de apoio à decisão, e não como única regra automática para bloqueio de transações. Em um ambiente real, seria necessário validar os impactos financeiros, operacionais e regulatórios antes da implantação.
