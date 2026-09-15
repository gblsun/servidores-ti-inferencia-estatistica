# Torre de Controle — Confiabilidade de Servidores em Nuvem

> *"Em SRE, você não evita as falhas. Você as vê chegando antes de todo mundo."*

Bem-vindo(a) à minha central de comando para o **Exercício Programa de Inferência Estatística**! Aqui
dentro eu coloco o chapéu de engenheiro de confiabilidade (SRE) por um dia, e uso estatística +
machine learning pra vasculhar 500 servidores em busca de sinais de que algo tá prestes a pegar fogo.

---

**Curso:** Ciência da Computação — 6º semestre — UNIMPACTA
**Disciplina:** Inferência Estatística · Prof. Msc. Kyung Moo Kim
**Autor:** Gabriel Muchon Pavanelli
**Modalidade:** Trabalho individual (nada de copiar, o professor avisou)
**Entrega:** 15 de setembro, via link do Google Colab

---

## A missão

Imagine que você é o plantonista de SRE de uma nuvem híbrida (AWS + Azure + On-Premise) e recebe um
dataset com a telemetria de 500 servidores. Seu trabalho é responder, com rigor estatístico — não no
"achismo" — três perguntas que todo time de infra adora fazer:

1. **Em média, quão sobrecarregada está a nossa frota de servidores?** (e o quão confiante posso estar
   nesse número?)
2. **O provedor de nuvem que eu escolho realmente muda minha chance de sofrer uma falha?**
3. **Dá pra prever, com machine learning, quais servidores estão prestes a falhar — e o que acontece se
   o modelo errar bem na hora que mais importa?**

Este repositório é a resposta a isso, documentada célula por célula.

## Como o repositório está organizado

```
.
├── README.md                          # você está aqui
├── requirements.txt                   # as engrenagens do projeto
├── .gitignore                         # o que fica de fora do controle de versão
├── data/
│   └── servidores_ti.csv              # os 500 servidores sob vigilância
└── notebooks/
    └── exercicio_programa_inferencia_estatistica.ipynb   # o EP completo, já executado
```

Tudo separado por responsabilidade: dado é dado, análise é análise, ninguém pisa no pé de ninguém.

## Stack usada

`pandas` para manipular os dados · `scipy.stats` para inferência · `scikit-learn` para PCA e KNN ·
`matplotlib` + `seaborn` para os gráficos que deixam tudo mais bonito de explicar.

## O que o notebook faz, parte por parte

### Parte 1 — Estimadores e Intervalo de Confiança
Calculo média e variância amostrais de `uso_cpu` e `latencia_rede_ms`, e construo um **IC de 95%** para
a média populacional de uso de CPU usando a distribuição t de Student.

> **Spoiler dos resultados:** X̄ = **55.11%** de uso de CPU, com IC 95% de **[53.78%, 56.44%]**. Ou seja,
> posso apostar com bastante confiança que a frota inteira roda, em média, na faixa dos 54–56% — nem
> ociosa, nem em pânico.

### Parte 2 — Teste Qui-Quadrado: o provedor de nuvem importa?
Cruzo `tipo_rede` (AWS / Azure / On-Premise) com `status_alerta` numa tabela de contingência e aplico o
teste Qui-Quadrado de independência (α = 0.05).

> **Spoiler dos resultados:** χ² = 0.0679, p-valor = **0.9666**. Ou seja: **não rejeito H₀** — nesta
> amostra, o provedor de nuvem escolhido não tem relação estatística com a chance de falha iminente. Pode
> escolher AWS, Azure ou On-Premise sem medo de que isso, sozinho, te condene a mais incidentes.

### Parte 3 — PCA: comprimindo o caos em 2 dimensões
Padronizo as 4 métricas contínuas e projeto tudo em 2 Componentes Principais (PC1 e PC2), pra visualizar
o "mapa de saúde" da frota inteira em um único gráfico.

> **Spoiler dos resultados:** PC1 explica **26.6%** da variância e PC2 mais **26.0%** — juntas, **52.6%**
> acumulados. Nada mal pra sair de 4 dimensões pra 2 sem perder metade da história.

### Parte 4 — KNN: colocando um vigia automático de plantão
Treino um classificador KNN (K=5) sobre PC1/PC2 (split 70/30, `random_state=42`) pra prever
`status_alerta`, avalio com matriz de confusão e relatório de classificação, e termino com uma análise
crítica sobre o pior tipo de erro que um SRE pode cometer: **o falso negativo**.

> **Spoiler dos resultados:** acurácia geral de **79%**, mas o recall da classe "Falha iminente" é de
> apenas **7%** — o modelo é ótimo em dizer "tá tudo bem" (às vezes bem otimista demais) e péssimo em
> pegar as falhas de verdade. É exatamente esse tipo de armadilha operacional que a análise crítica no
> notebook desmonta: acurácia alta não significa "seguro para produção".

## Como rodar essa nave

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook notebooks/exercicio_programa_inferencia_estatistica.ipynb
```

O notebook lê o CSV pelo caminho relativo `../data/servidores_ti.csv` — então execute a partir da pasta
`notebooks/` (ou mantendo a estrutura de pastas do repositório intacta) e todas as células rodam de
ponta a ponta sem erro.

## Entrega

A entrega oficial é o link público do Google Colab, com todo o código executado, saídas visíveis e os
comentários dissertativos em markdown. Basta subir o notebook (e o `servidores_ti.csv`, ajustando o
caminho de leitura se necessário) e compartilhar com permissão de visualização.

---

<sub>Feito com café, pandas (o da estatística, não o do zoológico) e uma pitada de paranoia de SRE por
Gabriel Muchon Pavanelli.</sub>
