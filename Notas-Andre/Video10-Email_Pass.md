# Email e Password
Este tutorial irá ensinar a você a desenvolver uma aplicação onde seja permitido que o usuário receba um e-mail para resetar sua senha.

## Token expirável com o tempo
Por questões de segurança precisamo aprender a gerar um __token que expire depois de um certo tempo__ (time sensitive token), desta forma somemte um usuário específico, e dentro de um prazo estabelevido, poderá resetar sua própia senha.... 
Assim que aprendermos a gerar esse tipo de Token expirável aprenderemos como enviar essa informação para o usuário usuário.

## Como gerar um Token sensível ao tempo (time sensitive)
precimos garantir que somente quem tenha acesso àquele e-mail específico possa resetar a sua senha
para fazer isso vamos usar um pacote chamado __itsdangerous__
esse pacote é normalmente instalado quando instalamos o Flask

## Entendo o via terminal
Vamos abrir um terminal python, 
> no caso do windows basta pressionar WIN + R e depois digitar CMD 
na linha de comando abra o ```Python```


```
>>> from itsdangerous import TimedJSONWebSignatureSerializer as Serializer 
```

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
Acabamos de gerar um TOKEN que vai expirar em 30 segundos

Para checar se esse token é válido podemos usar o método __loads__ 

###  Fora do prazo 
Se isso for feito em menos de 30 segundos vamos ver caso contrário receberemos uma mensagem:
> itsdangerous.exc.SignatureExpired: Signature expired

### Dentro do tempo
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
```
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
```
e vamos colar bem abaixo do __from datetime import datetime__
os módulos vão ficar mais ou menos assim:

```
from datetime import datetime
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flaskblog import db, login_manager
from flask_login import UserMixin
```

## Trazendo a chave secreta do app para o models.py
Nós também vamos precisar da chave secreta usada pelo nosso app.
Dessa forma, como nosso projeto se chama flaskblog, temos que importar dele
````
from flaskblog import db, login_manager, app 
````

dessa forma o nosso arquivo models vai ficar com os seguintes imports
````
from datetime import datetime
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flaskblog import db, login_manager, app
from flask_login import UserMixin
````
## Facilitando a criação dos tokens
agora dentro do User Model (class User) vamos criar alguns métodos que vão facilitar a criação dos Tokens
vamos chamar esse método de __get_reset_token()__

### Método de criaçao de token para reset
    def get_reset_token(self, expires_sec=1800):
        s = Serializer(app.config['SECRET_KEY'], expires_sec)
        return = s.dumps({'user_id': self.id}).decode('utf-8')

### Método para verificar o token gerado
    def verify_reset_token(token):
        # dessa vez não precisamos passar o tempo de expiração
        # esse tempo foi passado na criação do Token
        s = Serializer(app.config['SECRET_KEY']) 
        # Pode acontecer alguma exceção quando tentarmos carregar esse token
        # O Token pode ser inválido, ou o tempo já ter expirado
        # Ou qualquer coisa assim
        # Dessa forma vamos colocar um Try e Except Block
        try: #vamos tentar trazer o user id 
            user_id = s.loads(token)['user_id']
        except: # se houver uma exceção 
            return None 
        return User.query.get(user_id) # Caso não aconteça nenhuma exceção
    
Verificamos que esse método não faz nada com a instância do usuário
por exemplo, ele nunca usa a variável self
assistir a Object Oriented Series de MS COREY 
nós precisamos dizer ao python que se trata de método estático usando um decorator 
```@staticmethod``` acima do método...

Dessa forma o código fica assim (vou tirar os comentários ok?)
    
    @staticmethod
    def verify_reset_token(token):
        s = Serializer(app.config['SECRET_KEY']) 
        try: 
            user_id = s.loads(token)['user_id']
        except:
            return None 
        return User.query.get(user_id)

Com esse ```@staticmethod``` acima do método basicamente você está informando ao python que ele somente ele só vai trabalhar com o a variável token!

## Continuando...
Agora nós temos como gerar e validar tokens! 
Então precisamos:
1. criar as rotas para os usuários resetarem suas senhas...
2. aprender como enviar um email com um link necessário

Bem... ieremos precisar criar dois novos __forms__ para nossas routes...
Mas vão ser formulários extremamente simples!
Vamos abrir nosso __forms.py__

## Form 01 - Onde o usuário solicitará reset de senha
É um formulário simples.
Vamos chamar de __RequestResetForm__
O usuário irá informar o email e solicitará reset de senha

    class RequestResetForm(FlaskForm):
        email = StringField('Email',
                            validators=[DataRequired(), Email()])
        submit = SubmitField('Request Password Reset')

mas seria interessante que verificassemos se ele digitou certo, ou se aquele e-mail está no banco de dados... Caso contrário ele simplesmente precisa criar uma conta e não requisitar uma senha de reset.


    class RequestResetForm(FlaskForm):
        email = StringField('Email',
                            validators=[DataRequired(), Email()])
        submit = SubmitField('Request Password Reset')

        def validate_email(self, email):
            # ao invés de testar se o email existe, vamos ver se ele não existe
            if email.data != current_user.email:
                user = User.query.filter_by(email=email.data).first()
                if user is None:
                    raise ValidationError('There is no account with this email. You must register first.')

 ## Form 2 - Onde o usuário alterará a senha
 Serão dois campos de input
 1. Usuário digita a senha
 2. usuário digita a senha novamente em outro campo
 3. Clica em submit
 4. se as duas forem iguais haverá o reset de senha

        class ResetPasswordForm(FlaskForm):
            password = PasswordField('Password', validators=[DataRequired()])
            confirm_password = PasswordField('Confirm Password',
                                            validators=[DataRequired(), EqualTo('password')])
            submit = SubmitField('Reset Password')

# Passo 03 - Criar nossas rotas

## Importar os fomulários criados
1. Vamos abrir o arquivo __routes.py__
2. Vamos importar para o __routes.py__ os formulários que acabamos de criar, ou seja __RequestResetForm__ e  __ResetPasswordForm__

````
from flaskblog.forms import RegistrationForm, LoginForm, UpdateAccountForm, PostForm, RequestResetForm, ResetPasswordForm
````

Dentro do routes.py o arquivo ficará assim:

    import os
    import secrets
    from PIL import Image
    from flask import render_template, url_for, flash, redirect, request, abort
    from flaskblog import app, db, bcrypt
    from flaskblog.forms import RegistrationForm, LoginForm, UpdateAccountForm, PostForm, RequestResetForm, ResetPasswordForm
    from flaskblog.models import User, Post
    from flask_login import login_user, current_user, logout_user, login_required

# Criar as rotas
Vamos criar as sequintes rotas
__reset_request__: onde o usuário vai inserir o seu email e solicitar o token para mudar usuário

__reset_token__: onde o usuário vai trocar a senha, sendo o token ativo!

## reset_request
1. Não é necessário que ele esteja logado! então nada de @loginrequired
2. Mas se ele estiver logado e digitar reset password? Masdamos ele para home, ok?

Código da route então ...
```
@app.route("/reset_password", methods=['GET', 'POST'])
def reset_request():
    if current_user.is_authenticated:
        return redirect(url_for('home'))
    form render_template(reset_request.html, title='Reset Password', form=form)
```

mas precisamos verificar que se token enviado por e-mail está ativo certo?


## reset_token
Será onde o usuário vai entrar com a nova senha 
É necessário que o token esteja ativo
a url será algo tipo __/reset_password/<token>__ 


# Enviando email
## Extensão
Para enviar e-mail nós vamos usar uma outra extensão chamada de __flask-mail__.  Para isso precisaremos instalar ok?

## Instalando flask-mail
```pip install flask-mail```

## Importando flask-mail
Trazemos para o nosso arquivo __init.py__
```
from flask_mail import Mail
``` 

Assim o nosso __init.py__ está com os seguintes imports

```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt
from flask_login import LoginManager
from flask_mail import Mail
```

## Configurando mail server
agora precisamos configurar algumas coisas
Nós vamos precisar de um
- mail server
- mail port

de forma a usar TLS
- username e senha 

Para fins deste exemplo vou estar usando __GMAIL__  

Vamos escrever algumas linhas de código na parte de configurações do __init.py__, ok?
```
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
```
Agora para usuário e senha não vou digitar aqui por questões óbvias!  Só se eu quisesse que todo mundo tivesse acesso a meu e-mail...

Para este tipo de informação eu vou colocar como uma ___enviroment variable___

Para esconder informações sensíveis é necessário um tutorial específico.  

Para ler o tutorial sobre ___enviroment variable___ clique no link  https://github.com/andrenevares/Flask_Blog/blob/master/Notas-Andre/Enviroment-Variables.md

Caso queira ver um exemplo de código: https://github.com/andrenevares/codigos/blob/master/python/enviroment-variables/enviroment-variables.py

## ... Depois que você armazene suas enviroment variables 
No meu caso eu criei duas variáveis __EMAIL_USER__ e __EMAIL_PASSWORD__
o código ficará mais ou menos assim:

```
import os
...

app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('EMAIL_USER')
app.config['MAIL_PASSWORD'] = os.environ.get('EMAIL_PASSWORD')
```

## Iniciar a extensão
Após configurar a extensão você poderé inicia a extensão no seu código!
```
mail = Mail(app)
```

## O código final fica mais ou menos assim
```
import os
...

app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('EMAIL_USER')
app.config['MAIL_PASSWORD'] = os.environ.get('EMAIL_PASSWORD')
mail = Mail(app)
```

## Voltando ao Routes

Depois de no __init.py__:
1. importar o módulo ```import os```
2. fazer o app.config do ['MAIL_SERVER'], ['MAIL_PORT'], ['MAIL_USE_TLS'], ['MAIL_USERNAME'] e ['MAIL_PASSWORD'] 
3. e inicar a extensão por meio do ``` mail = Mail(app)```
... Vamos voltar ao __routes.py__

## Importando o mail para o routes
Como nossa aplicação se chama flaskblog vamos importar do flaskblog a instância mail criada no __init.py__
Mas também precisamos possibilitar o envio e isso será feito por meio __Message__ 
### Comando básico de import
```
from flaskblog import mail
from flask.mail import Message
```


###  Como estão nossos imports 
```
import os
import secrets
from PIL import Image
from flask import render_template, url_for, flash, redirect, request, abort
from flaskblog import app, db, bcrypt, mail
from flaskblog.forms import RegistrationForm, LoginForm, UpdateAccountForm, PostForm, RequestResetForm, ResetPasswordForm
from flaskblog.models import User, Post
from flask_login import login_user, current_user, logout_user, login_required
from flask.mail import Message
```

## A função de envio
```
def send_reset_email(user):
    token = user.get_reset_token()
    msg = Message('Assunto', sender='email@dequemenvia.com', recipients=[user.email])
    msg.body = f'''To reset your password, please visit the following link:
    {url_for('reset_token', token=token, _external=True)}
    
    If you did not make this request, then simple ignore this email and no change will be maid.
    '''
```
> Observação 01: Estamos usando a sintaxe de multi line que se inicia com f''' ... '''.  Isso permite que façamos uma mensagem simples no sistema.

> Observação 02: Podemos usar templates do Jinja2 mas ainda não fizemos nesse tutorial

> Observação 03: Usamos dentro do ```f''' {url_for()} '''``` com apenas uma chave... diferentemente da sintaxe usada no jinja2 quando usamos dois colchetes separados por espaços ```{{ url_for() }}``` 

> Observação 04: Usamos o ```_external=True``` ao invés de usar uma url estática, estaremos usando uma url relativa