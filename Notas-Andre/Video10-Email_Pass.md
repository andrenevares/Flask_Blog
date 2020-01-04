# Email e Password
Este tutorial irá ensinar a você a desenvolver uma aplicação onde seja permitido que o usuário receba um e-mail para resetar sua senha.

Primeiro vamos aprender a gerar um token time-sensitive de modo que somemte um usuário específico possa resetar a sua própia senha.... depois vamos ver como enviar para o usuário essa informação para o email do usuário.

## Como gerar um Token sensível ao tempo (time sensitive)
para garantir que somente uma pessoa que tenha acesso àquele e-mail específico possa resetar a sua senha
para fazer isso vamos usar um pacote chamado __itsdangerous__
esse pacote é normalmente instalado quando instalamos o Flask

Vamos abrir um terminal python
``` >>> from itsdangerous import TimedJSONWebSignatureSerializer as Serializer ```
Se você digitou corretamente não devemos ter nenhum tipo de erro

Agora nós vamos usar esse Serializer para passar:
1. uma chave secreta
2. um tempo de validade (em segundos)

```
>>> s = Serializer('secret', 30)
>>> token = s.dumps({'user_id': 1}).decode('utf-8')
>>> token
'eyJhbGciOiJIUzUxMiIsImlhdCI6MTU3ODE1OTIzMiwiZXhwIjoxNTc4MTU5MjYyfQ.eyJ1c2VyX2lkIjoxfQ.7G26p6OYxCb1SeXLuFZTuKg1P6JXppIrERcnGAu37EAOqTtYcA64TgMGXjLPKu7UwYX3g9tJ0VEX4E-njBazIg'
```
Para checar se esse token é válido podemos usar o método __load s__ 
Se isso for feito em menos de 30 segundos vamos ver caso contrário receberemos uma mensagem:
> itsdangerous.exc.SignatureExpired: Signature expired

Porém ao fazer isso antes dos 30 segundos que nós setamos iremos conseguir esse resultado:
```
>>> s.loads(token)
{'user_id': 1}
``` 

Isso é bom porque permite que nós lidemos com token e prazo de expiração sem ter que nos preocupar em colocar isso dentro do banco de dados e coisa e tal!

## Adicionar módulos ao User Model
Agora vamos voltar ao nosso projeto e vamos adicionar alguns módulos no nosso Modelo User Model
Se nós formos para o nosso projeto e abrirmos o arquivo __models.py__

### Importar Módulo TimedJSONWebSignatureSerializer
O primeiro passo seria importar os comandos que fizemos no prompt 
Então vamos pegar o
```
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
```
e vamos colar bem abaixo do __from datetime import datetime__
os módulos vão ficar mais ou menos assim:

```
from datetime import datetime
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flaskblog import db, login_manager
from flask_login import UserMixin```
```

Nós vamos precisar também da chave secreta usada no app
então vamos chamar a app do flaskblog ok 




