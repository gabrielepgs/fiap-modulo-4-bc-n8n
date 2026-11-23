# FIAP Pós Tech · Integração com APIs e Ferramentas Externas

Material prático do **Módulo 4 da Pós Tech em Agentes de IA**. Ao longo das aulas, você constrói no n8n um fluxo que estima despesas em dólar usando a PTAX do Banco Central do Brasil. A consulta inicial evolui para um processo que registra solicitações, devolve um protocolo e permite acompanhar o resultado.

**Docente:** Gabriele Guedes

## Aulas

| Aula | Prática |
| --- | --- |
| **1 · Consulta e cálculo** | Validar os dados, consultar a PTAX de venda e devolver uma estimativa rastreável pelo chat. |
| **2 · Processamento confiável** | Registrar pedidos via chat ou webhook, acompanhar por protocolo e testar reenvio, retry e reprocessamento. |

## Workflows

Os JSONs exportados ficam em `workflows/`. Use o guia prático para acompanhar a evolução de cada aula.

| Workflow | Onde entra | Para que serve |
| --- | --- | --- |
| `FIAP_A1_CONSULTAR` | Aula 1, com alterações na Aula 2 | Valida a entrada, consulta a PTAX e calcula a estimativa. Na Aula 2, recebe a política de tentativas. |
| `FIAP_A1_CHAT` | Aula 1, com alterações na Aula 2 | Usa as ferramentas do agente para conversar com o solicitante. Na Aula 2, registra e acompanha pedidos. |
| `FIAP_A2_RECEBER` | Aula 2 | Registra a solicitação em `fiap_pedidos` e devolve o protocolo. |
| `FIAP_A2_HTTP` | Aula 2 | Expõe o webhook de entrada para o envio do pedido. |
| `FIAP_A2_PROCESSAR` | Aula 2 | Processa um pedido por execução e grava a estimativa ou a falha. |
| `FIAP_A2_STATUS` | Aula 2 | Busca o estado e o resultado de um pedido pelo protocolo. |
| `FIAP_A2_REPROCESSAR` | Aula 2 | Reencaminha uma solicitação com falha quando ela ainda tem direito a outra execução. |

**Ao acompanhar as aulas, edite os workflows existentes e mantenha os nomes.** Crie outro workflow quando a prática introduzir uma nova função. Se já tiver montado um fluxo em uma aula anterior, aplique as alterações do guia no editor.

## Como importar no n8n

1. Use um ambiente n8n com **Data Tables** e um modelo de chat compatível com chamadas de ferramentas. Configure a credencial do modelo no próprio n8n.
2. Baixe o JSON. No editor de um workflow novo, abra o menu **⋯ → Import from File**. [Documentação de importação e exportação do n8n](https://docs.n8n.io/workflows/export-import/)
3. Na **Aula 1**, importe primeiro `FIAP_A1_CONSULTAR` e depois `FIAP_A1_CHAT`. No chat, selecione a consulta da sua instância na ferramenta do agente. Se o export do chat já estiver no estágio da Aula 2, use o guia da Aula 1 para montar a configuração inicial.
4. Na **Aula 2**, crie a Data Table `fiap_pedidos` conforme o guia. Importe ou construa primeiro `FIAP_A2_RECEBER` e `FIAP_A2_STATUS`. Depois configure o webhook, o processador e o reprocessamento. Aplique as mudanças nos workflows de consulta e chat que você já criou.
5. Nos nós **Data Table**, **Execute Sub-workflow** e **Call n8n Workflow Tool**, confira as referências. Selecione a tabela e os workflows da **sua instância**. Para o webhook, configure uma credencial **Header Auth** de teste com o cabeçalho `x-fiap-key` e copie a URL do seu próprio nó.

## Como testar

- **Aula 1.** No chat, peça: “Estime em reais US$ 800 de hospedagem em 10 de novembro de 2026.” Confira o valor, a data, a fonte e a chamada à ferramenta nos logs.
- **Aula 2.** Pelo webhook, envie um pedido com uma `chave_pedido` nova. Confira o `202 Accepted` e o protocolo. Consulte o status, rode o processador e consulte **o mesmo protocolo**. O estado deve passar de `recebida` para `concluida` quando o cálculo tiver sucesso.
- **Reenvio.** Envie de novo o mesmo conteúdo com a mesma chave. O retorno esperado é o protocolo existente. Alterar o conteúdo mantendo a chave deve gerar `409`.
- **Falhas.** Siga a simulação de `503` no guia para observar o retry. Para testar o reprocessamento, use um pedido de teste em `falha` e confira que o protocolo é preservado.

## Antes de compartilhar um JSON

Revise cada export e retire **chaves de API, tokens, cabeçalhos de autenticação preenchidos, dados reais e dados fixados no editor**. Anonimize referências de credenciais e confira se ficou algum endereço interno. Quem importar configura a própria credencial no n8n. [Orientações do n8n para compartilhar workflows](https://docs.n8n.io/workflows/export-import/)
