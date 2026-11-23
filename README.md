# Food Hybrid Search Engine

[![CI Tests](https://github.com/itamacdo/food-hybrid-search/actions/workflows/ci.yml/badge.svg)](https://github.com/itamacdo/food-hybrid-search/actions/workflows/ci.yml)

Motor de busca semântica e substituição alimentar que combina **recuperação vetorial (Dense Retrieval)** com **filtragem determinística tabular (Pandas)**, executado localmente sem dependência de APIs externas pagas.

## Problema Resolvido

Buscas puramente vetoriais em recomendações alimentares podem ignorar restrições críticas (como alergênicos, intolerância à lactose ou limites calóricos e financeiros) quando há proximidade semântica no texto. Por outro lado, filtros tabulares isolados não interpretam intenções em linguagem natural (ex: *"substituto acessível para salmão com perfil proteico similar"*).

A arquitetura **Hybrid Search** resolve ambas as limitações aplicando pré-filtragem estrita sobre metadados nutricionais antes do ranqueamento por similaridade de cosseno.

## Arquitetura do Fluxo

1. **Indexação Vetorial:** Geração de embeddings densos normalizados em L2 utilizando o modelo multilíngue `paraphrase-multilingual-MiniLM-L12-v2` (`sentence-transformers`).
2. **Pré-Filtragem Determinística:** Aplicação de regras rígidas via `Pandas` para tags obrigatórias (`sem_lactose`, `vegano`), exclusão de restrições, piso proteico (`min_protein_g`), teto calórico (`max_calories`) e faixa de custo (`max_cost_tier`).
3. **Ranqueamento Semântico:** Cálculo de produto escalar (`NumPy`) entre o vetor da consulta e os candidatos aprovados no filtro determinístico.
4. **Contrato REST:** Validação estrita de entrada e saída via `FastAPI` e `Pydantic`.

## Estrutura do Projeto

```text
food-hybrid-search/
├── .github/workflows/ci.yml   # Pipeline de integração contínua (Pytest)
├── data/foods.json            # Base tabular de alimentos e metadados nutricionais
├── src/
│   ├── main.py                # Endpoints REST (FastAPI)
│   ├── schemas.py             # Contratos de entrada e saída (Pydantic)
│   └── search_engine.py       # Motor de filtragem determinística e busca vetorial
├── tests/
│   └── test_search.py         # Testes automatizados de restrições e ranqueamento
└── requirements.txt           # Dependências do projeto
```

## Stack

- **Python 3.11+**
- **Sentence-Transformers & PyTorch (CPU)**
- **Pandas & NumPy**
- **FastAPI & Pydantic**
- **Pytest & GitHub Actions**

## Exemplo de Uso

**Requisição (`POST /search`):**

```json
{
  "query": "substituto acessível para salmão rico em proteína e ômega 3",
  "required_tags": ["sem_lactose"],
  "min_protein_g": 20.0,
  "max_cost_tier": 1,
  "top_k": 2
}
```

**Resposta:**

```json
{
  "query": "substituto acessível para salmão rico em proteína e ômega 3",
  "total_candidates_after_filter": 2,
  "results": [
    {
      "id": "food_02",
      "name": "Sardinha Assada",
      "category": "Peixes e Frutos do Mar",
      "description": "Alternativa acessível de peixe fonte de ômega-3, cálcio e proteínas completas, excelente substituto nutricional para peixes nobres.",
      "calories_100g": 164.0,
      "protein_g": 24.6,
      "carbs_g": 0.0,
      "fat_g": 6.8,
      "cost_tier": 1,
      "tags": ["sem_gluten", "sem_lactose", "rico_em_proteina", "economico", "almoco_jantar"],
      "semantic_score": 0.7842
    }
  ]
}
```
