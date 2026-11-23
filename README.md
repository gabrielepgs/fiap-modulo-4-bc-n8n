# FIAP Pós Tech · Integração com APIs e Ferramentas Externas

Material prático do **Módulo 4 da Pós Tech em Agentes de IA**. O projeto usa o **n8n** e a **API PTAX do Banco Central do Brasil** para estimar despesas em dólar. Ao longo de quatro aulas, a consulta inicial evolui para um processo com protocolo, tratamento de falhas, aprovação humana e auditoria.

**Docente:** Gabriele Guedes

## Aulas

| Aula | Tema | O que você vai construir |
| --- | --- | --- |
| **1** | Padrões de integração para agentes de IA | Um agente que valida despesas, consulta a PTAX de venda e devolve a estimativa com data e fonte. |
| **2** | Webhooks, eventos e integrações confiáveis | Recebimento via chat ou webhook, processamento por protocolo e tratamento de reenvios, retries e reprocessamento. |
| **3** | Automação de workflows com agentes | Uma política de aprovação com limite didático de R$ 6.000, formulário de decisão, expiração e finalização. |
| **4** | Segurança e controle na execução de ações por agentes | Credenciais separadas, ferramentas com escopo definido e uma trilha de auditoria para as transições do pedido. |

## Workflows

Use a pasta `workflows/` para os JSONs exportados. Siga os guias práticos na ordem das aulas. Quando uma aula ampliar uma função existente, **edite o workflow que você já criou e mantenha o nome**. Crie um workflow quando a aula introduzir uma nova função.

| Workflow | Aula de criação | Para que serve |
| --- | --- | --- |
| `FIAP_A1_CONSULTAR` | 1 | Valida a entrada, consulta a PTAX e calcula a estimativa. Recebe a política de tentativas na Aula 2. |
| `FIAP_A1_CHAT` | 1 | Recebe pedidos pelo chat e chama as ferramentas do agente. Na Aula 2, registra e acompanha solicitações. |
| `FIAP_A2_RECEBER` | 2 | Registra a solicitação em `fiap_pedidos` e devolve o protocolo. |
| `FIAP_A2_HTTP` | 2 | Recebe pedidos por webhook e encaminha ao workflow de registro. |
| `FIAP_A2_PROCESSAR` | 2 | Processa o pedido e grava a estimativa ou a falha. Na Aula 3, aplica a política de aprovação. |
| `FIAP_A2_STATUS` | 2 | Busca o estado e o resultado de um pedido pelo protocolo. |
| `FIAP_A2_REPROCESSAR` | 2 | Reencaminha pedidos com falha que ainda podem ser processados novamente. |
| `FIAP_A3_APROVAR` | 3 | Abre um formulário protegido para registrar a aprovação ou a rejeição. |
| `FIAP_A3_EXPIRAR` | 3 | Marca como expirados os pedidos que ficaram sem decisão até o prazo. |
| `FIAP_A3_FINALIZAR` | 3 | Retoma a finalização de uma solicitação cuja decisão já foi registrada. |
| `FIAP_A4_AUDITAR` | 4 | Grava eventos de transição em `fiap_auditoria`, relacionados ao protocolo e à execução. |

## Como importar e configurar

1. Use um ambiente n8n com **Data Tables** e configure uma credencial para um modelo de chat compatível com chamadas de ferramentas.
2. Baixe o JSON do workflow que você vai criar. No editor do n8n, abra **⋯ → Import from File**. Após importar, selecione nas ferramentas e nos nós de subworkflow as referências da sua própria instância. [Documentação de importação e exportação do n8n](https://docs.n8n.io/workflows/export-import/)
3. **Aula 1.** Importe ou monte `FIAP_A1_CONSULTAR` antes de configurar `FIAP_A1_CHAT`. No chat, confira se a ferramenta aponta para a consulta.
4. **Aula 2.** Crie a Data Table `fiap_pedidos` conforme o guia. Monte o recebimento e o status antes de configurar os fluxos que os chamam. No webhook de entrada, selecione uma credencial de teste **Header Auth** com o cabeçalho `x-fiap-key` e use a URL do seu nó no cliente HTTP.
5. **Aula 3.** Reutilize `fiap_pedidos` e acrescente a política ao `FIAP_A2_PROCESSAR`. Configure o formulário com uma credencial **Basic Auth** exclusiva de aprovador. Use os workflows de expiração e finalização para testar o ciclo depois da decisão.
6. **Aula 4.** Crie a Data Table `fiap_auditoria`. Configure `FIAP_A4_AUDITAR` e conecte-o aos pontos de transição indicados no guia. Confira as referências de tabela e de subworkflow em cada nó.

## Testes de ponta a ponta

| Aula | O que conferir |
| --- | --- |
| **1** | Peça no chat uma estimativa de **US$ 800 de hospedagem em 10/11/2026**. Confira no resultado o valor, a data e a fonte. Confirme nos logs que o agente chamou a ferramenta. |
| **2** | Envie um pedido novo e confira o `202 Accepted`. Consulte o mesmo protocolo antes e depois do processamento. Reenvie o mesmo conteúdo com a mesma chave e confira o protocolo existente. Altere o conteúdo mantendo a chave e confira o `409`. |
| **3** | Teste uma estimativa até **R$ 6.000** e outra acima do limite. Confira `dentro_da_politica` no primeiro caso e `aguardando_aprovacao` no segundo. Teste também aprovação, rejeição, expiração e retomada da finalização. |
| **4** | Teste o acesso ao formulário com a credencial do solicitante e confirme a recusa. Execute uma transição permitida e confira o evento em `fiap_auditoria` pelo protocolo. |

## Antes de compartilhar um JSON

Revise os exports e remova **chaves de API, tokens, cabeçalhos de autenticação preenchidos, dados reais e dados fixados no editor**. Anonimize referências de credenciais e revise endereços internos. Cada aluno configura suas próprias credenciais no n8n. [Orientações do n8n para compartilhar workflows](https://docs.n8n.io/workflows/export-import/)
