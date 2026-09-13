# apolo-adf

Repositório dos pipelines do Azure Data Factory — parte do projeto de portfólio [apolo-infra](https://github.com/papolonio/apolo-infra).

## Como este repositório é populado

Diferente dos outros dois repos do projeto, este **não recebe código escrito manualmente**. O Azure Data Factory tem uma funcionalidade nativa de **Git integration** (Manage → Git configuration, no ADF Studio): depois que a instância do ADF (provisionada via Bicep no repo `apolo-infra`) for conectada aqui, cada pipeline/dataset/linked service criado **pela UI normal do ADF Studio** é serializado automaticamente como JSON e commitado neste repo.

Fluxo esperado, uma vez conectado:

1. Editar pipelines na UI do ADF Studio, numa feature branch.
2. Cada "Save" gera commit automático neste repo.
3. "Publish" no ADF Studio gera/atualiza uma branch `adf_publish` com o ARM template pronto pra deploy — essa branch é a que o Linked Template Deployment de fato usa.

## Estado atual

Conectado e em uso. Recursos criados via UI do ADF Studio e sincronizados aqui:

- `linkedService/ls_adls_apolo.json` — conexão com o ADLS Gen2 via Managed Identity (sem credencial explícita)
- `dataset/ds_source.json` / `dataset/ds_landing.json` — datasets Binary parametrizados (`folderPath`, `fileName`), apontando para os containers `source` e `landing`
- `pipeline/pl_copy_source_to_landing.json` — pipeline com um `ForEach` iterando uma lista fixa de 6 arquivos fictícios (3 tabelas × CSV/Parquet), copiando cada um de `source/` para `landing/` via `Copy data`

### Nota: bug conhecido no botão "Publish" do ADF Studio

Na primeira publicação depois de conectar a Git integration, o botão **Publish** falhou com o erro `Cannot read properties of undefined (reading '__LAST_PUBLISHED_COMMIT_ID___')` — um bug de estado local do navegador (a definição já estava salva corretamente na branch `main` via Git integration; só a aplicação na factory ao vivo, que o Publish normalmente faz, não aconteceu). Refresh e aba anônima não resolveram.

**Workaround aplicado:** os arquivos JSON já commitados em `main` (`linkedService/`, `dataset/`, `pipeline/`) foram aplicados diretamente na factory via API REST do ARM (`PUT .../linkedservices/{name}`, `.../datasets/{name}`, `.../pipelines/{name}`, api-version `2018-06-01`), o que tem o mesmo efeito prático de "Publish" para uso imediato (a factory passa a ter os recursos ao vivo e disparáveis). A branch `adf_publish` (usada pra promover essa definição pra outro ambiente via CI/CD) ainda não foi gerada — só importa quando isso for necessário, o que não é o caso enquanto só existe uma factory de dev.

Validado com uma execução real (não Debug) do pipeline via API (`createRun`), resultado `Succeeded`, confirmando os 6 arquivos copiados de `source/` para `landing/`.

## Repositórios do projeto

- [apolo-infra](https://github.com/papolonio/apolo-infra) — infraestrutura (Bicep) e setup do Databricks
- **apolo-adf** (este repo) — pipelines do ADF
- [apolo-dbt](https://github.com/papolonio/apolo-dbt) — projeto dbt
