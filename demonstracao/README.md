# Pasta: demonstracao

Esta pasta contém imagens que documentam visualmente o fluxo *Step Function → Lambda → SNS* e explicações passo-a-passo para cada captura.

> Arquivos:
> - 01-stepfunction-diagram.png — Diagrama do fluxo no console do Step Functions  
> - 02-stepfunction-console-definition.png — Definição (ASL) da State Machine no console  
> - 03-sns-subscription-details.png — Detalhes da assinatura do tópico SNS (endpoint confirmado)  
> - 04-sns-email-notification.png — Exemplo de e-mail enviado pelo SNS com a mensagem JSON

---

## 01 — Diagrama do Workflow (01-stepfunction-diagram.png)

![Diagrama do Step Function](./01-stepfunction-diagram.png)

*O que mostra:*  
Visão gráfica do fluxo: Start → Lambda Invoke → SNS Publish → End.  

1. *Start:* início da execução da State Machine.  
2. *Lambda Invoke (Task State):* a máquina chama a função AWS Lambda (executa rotina).  
3. *SNS Publish (Task State):* após a Lambda, a Step Function publica uma mensagem no tópico SNS.  
4. *End:* fim da execução.

---

## 02 — Definição / Console (02-stepfunction-console-definition.png)

![Definição da State Machine](./02-stepfunction-console-definition.png)

*O que mostra:*  
Tela do console com a definição (JSON/ASL) da sua State Machine ao lado da visualização.  

*O que procurar no JSON:*
- "Resource": "arn:aws:states:::lambda:invoke" → indica invocação da Lambda.  
- "Resource": "arn:aws:states:::sns:publish" → indica publicação no SNS.  
- "Message" ou parâmetros execucaoId.$ / inicioExecucao.$ → ver como você compõe a mensagem enviada pelo SNS (ex.: $$.Execution.Id).

*Passo-a-passo para auditoria:*
1. Confirme o FunctionName passado para a task Lambda.  
2. Confirme o TopicArn usado na task SNS.  
3. Verifique se End: true está na última task.

---

## 03 — Detalhes da assinatura do SNS (03-sns-subscription-details.png)

![Detalhes da Assinatura SNS](./03-sns-subscription-details.png)

*O que mostra:*  
Tela do SNS com detalhes da *assinatura* — o endpoint (e-mail) está confirmado e o protocolo é EMAIL.

*Por que é importante:*  
- *Assinatura confirmada* é necessária para receber e-mails do tópico.  
- Verifique o Endpoint (seu e-mail) e o ARN da assinatura para auditoria.  
- Caso o e-mail não chegue: confirme status e checar spam, ou verifique se a política do tópico permite publicação pela entidade (role) que executa a Step Function.

---

## 04 — Exemplo do e-mail de notificação (04-sns-email-notification.png)

![Exemplo de Email SNS](./04-sns-email-notification.png)

*O que mostra:*  
E-mail recebido pelo SNS com o corpo contendo um JSON como:
```json
{
  "status":"SUCESSO",
  "mensagem":"Processo concluído com sucesso!",
  "execucaoId":"$$.Execution.Id",
  "inicioExecucao":"$$.Execution.StartTime"
}
