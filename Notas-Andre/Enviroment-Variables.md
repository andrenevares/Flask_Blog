# Enviroment Variables
Esse será um tutorial de como esconder informações secretas ou semsíveis como senhas e usuários e acessar essas informações através do seu código python.

__Não vá colocar informações no seu código que não deveriam estar no seu código!!!__

## O Problema
Se você estiver trabalhando com mais de uma pessoa, ou mais de uma pessoa tiver acesso ao seu código... se essas informações estiverem explícitas então você ficará de certa forma vulnerável...

Mas... se você armazenar essas informações dentro de __enviroment variables__ você continuará podendo compartilhar o seu código, mas sem expor essas informações que vão ficar armzenadas localmente.


> Observação:  Enviroment variables funcionam diferentemente no Windows e Mac.  Como no dia de hoje sou um usuário Windows, esse tutorial será para usuários windows!

## Criando um Enviroment Variable
1. Clique no ícone do windows
2. Vá até o __Painel de Controle__
3. Vá até __Sistema__
4. Vá até __Configurações Avançadas do sistema__
5. No canto direito da janela tem um botão __Variáveis de Ambiente__
6. Clique no botão __Novo__
7. Digite o nome da variável e valor que você quer atribuir
8. Após você adcionar os valores e variáveis, precisará clicar em OK para salvar.

> Observação: Você precisará fechar a sua IDE ou seu editor de texto e abrir novamente para que esses valores possam ser reconhecidos ok?

## Como usar uma enviroment variable no seu código


### Passo 01 - Importar o sistema operacional
Você precisará fazer o import do sistema operacional na primeira linha do seu código python por meio do ```import os```

### Passo 02 - Colocando no código

Ex. enviroment-variables.py


```
import os

email_user = os.environ.get('EMAIL_USER')
email_password = os.environ.get('EMAIL_PASS')

print(email_user)
print(email_password)
```





