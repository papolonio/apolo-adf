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
- `linkedService/ls_keyvault_apolo.json` — conexão com o Key Vault (Managed Identity)
- `linkedService/ls_function_bridge.json` — conexão com a Azure Function de ponte pro Databricks (`apolo-infra/functions/bridge_to_databricks/`), autenticação por chave de função lida do Key Vault (`ls_keyvault_apolo` → secret `BridgeFunctionKey`)
- `dataset/ds_source.json` / `dataset/ds_landing.json` — datasets Binary parametrizados (`folderPath`, `fileName`), apontando para os containers `source` e `landing`
- `pipeline/pl_copy_source_to_landing.json` — pipeline com um `ForEach` iterando uma lista fixa de 6 arquivos fictícios (3 tabelas × CSV/Parquet). Dentro de cada iteração: `Copy data1` (`source/` → `landing/`) seguido de `Bridge to Databricks` (Azure Function activity, empurra o mesmo arquivo do `landing` pro Volume `bronze.landing.raw_files` no Databricks)

### Nota: bug conhecido no botão "Publish" do ADF Studio

Na primeira publicação depois de conectar a Git integration, o botão **Publish** falhou com o erro `Cannot read properties of undefined (reading '__LAST_PUBLISHED_COMMIT_ID___')` — um bug de estado local do navegador (a definição já estava salva corretamente na branch `main` via Git integration; só a aplicação na factory ao vivo, que o Publish normalmente faz, não aconteceu). Refresh e aba anônima não resolveram.

**Workaround aplicado:** os arquivos JSON já commitados em `main` (`linkedService/`, `dataset/`, `pipeline/`) foram aplicados diretamente na factory via API REST do ARM (`PUT .../linkedservices/{name}`, `.../datasets/{name}`, `.../pipelines/{name}`, api-version `2018-06-01`), o que tem o mesmo efeito prático de "Publish" para uso imediato (a factory passa a ter os recursos ao vivo e disparáveis). A branch `adf_publish` (usada pra promover essa definição pra outro ambiente via CI/CD) ainda não foi gerada — só importa quando isso for necessário, o que não é o caso enquanto só existe uma factory de dev.

Validado com uma execução real (não Debug) do pipeline via API (`createRun`), resultado `Succeeded`, confirmando os 6 arquivos copiados de `source/` para `landing/`.

### Segunda nota: "Publish" pode aplicar na factory sem sincronizar o Git

Ao adicionar a Activity da Function (Fase 3), o Publish "funcionou" no sentido de aplicar as mudanças na factory ao vivo (confirmado via `GET` na API: linked services e pipeline atualizados de verdade), mas **não gerou nenhum commit novo** em `main` nem em `adf_publish` — ou seja, o Git ficou desatualizado mesmo com o Publish "dando certo" do ponto de vista funcional. Isso é diferente do bug anterior (que falhava visivelmente); aqui o Publish simplesmente não escreveu no repositório, silenciosamente.

**Workaround aplicado:** as definições foram lidas direto da factory ao vivo via `GET` na API do ARM e commitadas manualmente em `main`, pra manter o repositório como fonte de verdade fiel ao que está rodando de fato. Vale conferir a sincronização Git↔factory sempre que o Publish for usado neste projeto, em vez de assumir que funcionou por completo.

## Repositórios do projeto

- [apolo-infra](https://github.com/papolonio/apolo-infra) — infraestrutura (Bicep) e setup do Databricks
- **apolo-adf** (este repo) — pipelines do ADF
- [apolo-dbt](https://github.com/papolonio/apolo-dbt) — projeto dbt
