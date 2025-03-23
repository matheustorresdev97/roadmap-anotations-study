https://github.com/PicPay/picpay-desafio-backend


Tipos de Usuários - Comum - Lojista

Entities - Usuario,
- Nome Completo
- CPF / CNPJ - Único
- Email - Único
- Senha
- Relação com carteira
- Relação com transferência
- Tipo de Usuário


Carteira, Transferência


Regra de Negócio - 
Transferência de Dinheiro - Entre usuários e lojistas ou somente usuários
Lojistas somente recebem - Não podem fazer transferências

Validar saldo do usuário

Validar se a transferência foi autorizada (GET) -  https://util.devi.tools/api/v2/authorize

@Transactional = reerter a operação em caso de qualquer falha

Enviar email ao usuario que recebeu o pagamento(POST) - https://util.devi.tools/api/v1/notify

https://github.com/matheustorresdev97/transacao-simplificada