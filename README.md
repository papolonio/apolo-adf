# apolo-adf

Repositório dos pipelines do Azure Data Factory — parte do projeto de portfólio [apolo-infra](https://github.com/papolonio/apolo-infra).

## Como este repositório é populado

Diferente dos outros dois repos do projeto, este **não recebe código escrito manualmente**. O Azure Data Factory tem uma funcionalidade nativa de **Git integration** (Manage → Git configuration, no ADF Studio): depois que a instância do ADF (provisionada via Bicep no repo `apolo-infra`) for conectada aqui, cada pipeline/dataset/linked service criado **pela UI normal do ADF Studio** é serializado automaticamente como JSON e commitado neste repo.

Fluxo esperado, uma vez conectado:

1. Editar pipelines na UI do ADF Studio, numa feature branch.
2. Cada "Save" gera commit automático neste repo.
3. "Publish" no ADF Studio gera/atualiza uma branch `adf_publish` com o ARM template pronto pra deploy — essa branch é a que o Linked Template Deployment de fato usa.

## Estado atual

Ainda não conectado — o ADF só será provisionado na Fase 2 do roadmap (ver `IMPLEMENTATION_PLAN.md` no repo `apolo-infra`). Este repo existe desde já só pra reservar o nome/local que a Git integration vai usar.

## Repositórios do projeto

- [apolo-infra](https://github.com/papolonio/apolo-infra) — infraestrutura (Bicep) e setup do Databricks
- **apolo-adf** (este repo) — pipelines do ADF
- [apolo-dbt](https://github.com/papolonio/apolo-dbt) — projeto dbt
