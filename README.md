# Integração de Leads GrupoZap Versão 2.0

Este documento descreve a forma de integração de leads que o GrupoZap está disponibilizando para nossos clientes de forma que eles possam receber os leads enviados pelos usuários do portal diretamente em seus sistemas.

Endpoint:

https://yourdomain.com.br/lead/ID-DO-ANUNCIANTE

Ex:
https://crm.com.br/lead/123456

OU

https://ID-DO-ANUNCIANTE.yourdomain.com.br/lead

Ex:
https://imobiliariaxpto.crm.com.br/lead

ID-DO-ANUNCIANTE é parte da URL que representa o identificador do anunciante no sistema que irá receber a requisição. É uma forma de identificação do cliente xpto no CRM.

Caso não tenha ou não queira especificar o ID-DO-ANUNCIANTE o endpoint poderá ser implementado sem o identificador do anunciante. 

Ex: 
https://yourdomain.com.br/lead

Utilizando um único endpoint, o CRM deverá identificar o cliente e fazer a devida distribuição do lead a partir do dados que enviamos no body da requisição, como por exemplo o `clientListingId` que corresponde ao identificador do anúncio definido por nosso cliente (ExternalID).

Os leads serão enviados através de um callback que será feito em uma URL (endpoint) escolhida pelo cliente e que terá como payload os dados do lead. Esse endpoint deverá ser implementado de forma que entenda e processe o payload que será enviado e, uma vez que o lead seja recebido com sucesso, retorne um código de status HTTP adequado.

O payload será enviado no corpo do request através do método POST do protocolo HTTP no formato JSON e terá a seguinte estrutura:

Ex:
```
{
  "leadOrigin": "VivaReal",
  "timestamp": "2017-10-23T15:50:30.619Z",
  "originLeadId": "59ee0fc6e4b043e1b2a6d863",
  "originListingId": "87027856",
  "clientListingId": "a40171",
  "name": "Nome Consumidor",
  "email": "nome.consumidor@email.com",
  "ddd": "11",
  "phone": "999999999",
  "phoneNumber": "11999999999",
  "message": Olá, tenho interesse neste imóvel. Aguardo o contato. Obrigado.",
}
```

Onde:

- **leadOrigin**: Identificador do portal que o lead foi gerado (VivaReal, Zap);
- **timestamp**: Data e horário da criação do lead no formato ISO_LOCAL_DATE_TIME;
- **originLeadId**: Identificador do lead do GrupoZap;
- **originListingId**: Identificador do anúncio do GrupoZap;
- **clientListingId**: Identificador do anúncio para o anunciante (ExternalId);
- **name**: Nome do consumidor que gerou o lead;
- **email**: E-Mail do consumidor que gerou o lead;
- **ddd**: DDD do telefone do consumidor que gerou o lead Ex:`11`;
- **phone**: Telefone do consumidor que gerou o lead Ex:`999999999`;
- **phoneNumber**: [deprecado] Telefone do consumidor que gerou o lead (DDD + Phone) Ex:`11999999999`;
- **message**: Mensagem do consumidor que gerou o lead;

### Obs:

Os campos **phoneNumber**, **ddd** e **phone** são opcionais e podem não ser enviados na integração.
O campo phoneNumber está deprecado e será removido futuramente.

### leadOrigin
- `Zap`: Lead gerado no portal https://zapimoveis.com.br
- `VivaReal`: Lead gerado no portal https://vivareal.com.br

Ou seja, não será necessário criar 2 endpoints individualmente para cada portal, somente interprete o campo `leadOrigin` e faça a distribuição do lead a partir do portal específico.

O processo de integração será acionado para cada lead individualmente, sempre que recebermos um contato do usuário. Nosso controle de sucesso do recebimento dos leads será feito através dos [códigos de status](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) do protocolo HTTP da seguinte forma:

2xx: indica que o cliente recebeu o lead com sucesso e que ações adicionais não são necessárias. Qualquer código de status da família 200 terá esse efeito.
3xx, 4xx ou 5xx: indica que houve falha no processamento do lead por parte do cliente.

Caso o endpoint do cliente retorne qualquer código de status que não os da família 200, haverá retentativa automática no envio do lead. O reenvio será tentado por 6 vezes em intervalos de 30 minutos e após isso ele será armazenado temporariamente por até 14 dias, podendo ser reenviado a pedido do cliente.

O controle do status de recebimento dos leads será feito exclusivamente através dos códigos de status do protocolo HTTP, qualquer informação enviada no corpo da resposta será totalmente ignorada.

### Timeout
A requisição POST para o endpoint está configurada com timeout de 5 segundos, ou seja, qualquer requisição que demorar mais que 5 segundos será considerada ERRO sendo reenviadas de acordo com nossas regras de retentativas.

### Segurança

Para verificar a confiabilidade do emissor das requisições, será enviado no Header uma chave de segurança no padrão [Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication). A chave a ser verificada será no seguinte formato:

```
Authorization: Basic dml2YXJlYWw6U0VDVVJJVFktS0VZ
Decodifique a chave dml2YXJlYWw6U0VDVVJJVFktS0VZ com Base64 terá o valor
vivareal:SECRET-KEY
Valide a SECRET-KEY
```

A **SECRET-KEY** poderá ser solicitada para nossa equipe na adesão da integração.
Caso a SECRET-KEY não coincida com a chave que enviamos deverá retornar httpStatusCode 401

### Testes
Assim que as implementações forem devidamente realizadas e estiver pronto para iniciar os testes, entre em contato com: <p><a href="mailto:integracaoleads@grupozap.com">integracaoleads@grupozap.com</a></p>

### Dúvidas Sugestões ou Problemas
Caso tenha alguma dúvida, sugestão ou problemas durante a implementação da integração de Leads, abra um [Issue](https://github.com/grupozap/crm-lead-integration/issues) neste repositório que iremos responder assim que possível.

