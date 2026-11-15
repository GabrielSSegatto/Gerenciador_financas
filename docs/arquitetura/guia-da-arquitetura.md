# MoneyFlow — Guia da arquitetura do projeto

Guia da equipe · novembro de 2026 · Django com templates, PostgreSQL no Docker

## 1. O projeto em poucas linhas

O MoneyFlow é um sistema web para a pessoa controlar o dinheiro em um lugar só: contas bancárias, receitas e despesas (o caixa) e investimentos (ações, FIIs, criptomoedas e renda fixa). No fim, o sistema mostra o patrimônio total e como ele evolui ao longo do tempo.

Três ideias guiam todo o código:

- **Cada usuário só enxerga os próprios dados.**
- **Todo dinheiro que se move passa por uma conta bancária.** Uma despesa tira da conta, a compra de um ativo também tira, e uma venda devolve.
- **Saldo e histórico não podem ficar inconsistentes.** Quando algo muda o saldo, tudo acontece junto ou nada acontece.

## 2. Como rodar o projeto

Antes de começar, instale: **Git**, **Python 3.12 ou superior** e **Docker** (Docker Desktop no Windows; Docker Engine com o plugin Compose no Linux).

### Primeira vez no Windows (PowerShell)

```powershell
git clone https://github.com/GabrielSSegatto/Gerenciador_financas.git
cd Gerenciador_financas
docker compose up -d
cd src
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
Copy-Item .env.example .env
python manage.py migrate
python manage.py runserver
```

### Primeira vez no Linux (bash)

```bash
git clone https://github.com/GabrielSSegatto/Gerenciador_financas.git
cd Gerenciador_financas
docker compose up -d
cd src
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python manage.py migrate
python manage.py runserver
```

Depois abra `http://127.0.0.1:8000`. Enquanto as telas não existirem, aparece só a página padrão do Django (o foguete). Para parar o servidor, use `Ctrl+C`.

O que cada comando faz:

| Comando | O que faz |
|---|---|
| `docker compose up -d` | Sobe o PostgreSQL em segundo plano (rodar na raiz do repositório) |
| `python -m venv venv` | Cria a venv, uma caixa isolada de bibliotecas só deste projeto |
| ativar a venv | Faz o terminal usar essa caixa (aparece `(venv)` no começo da linha) |
| `pip install -r requirements.txt` | Instala as bibliotecas nas versões exatas do projeto |
| copiar o `.env.example` para `.env` | Cria suas variáveis locais (senha do banco, chave secreta) |
| `python manage.py migrate` | Cria as tabelas no banco |
| `python manage.py runserver` | Liga o site em modo de desenvolvimento |

### No dia a dia (depois da primeira vez)

No Windows:

```powershell
docker compose up -d          # na raiz do repositório
cd src
venv\Scripts\activate
python manage.py runserver
```

No Linux:

```bash
docker compose up -d          # na raiz do repositório
cd src
source venv/bin/activate
python manage.py runserver
```

Depois de cada `git pull`, rode também `pip install -r requirements.txt` e `python manage.py migrate`, para pegar bibliotecas e tabelas novas.

### Superusuário (opcional)

O `python manage.py createsuperuser` cria uma conta de administrador para entrar no painel `/admin`. Ele **não é necessário para rodar o sistema**, porque os usuários comuns se cadastram pela tela de cadastro do app accounts. Use só se precisar do painel (para conferir dados, por exemplo). Lembre que o banco de cada pessoa é local: o superusuário de um colega não existe na sua máquina.

### Problemas comuns

| Problema | O que fazer |
|---|---|
| Erro de conexão com o Docker no Windows | Abra o Docker Desktop e espere aparecer "Engine running" antes do `docker compose up -d` |
| Erro de permissão do Docker no Linux | Rode `sudo usermod -aG docker $USER` e saia e entre na sessão de novo (ou use `sudo` antes do comando) |
| `python3 -m venv` reclama no Linux | Instale o pacote com `sudo apt install python3-venv` (Debian e Ubuntu) |
| Porta 5432 ocupada | Você já tem um PostgreSQL instalado: desligue o local ou avise o grupo para mudar a porta no `docker-compose.yml` |
| `ModuleNotFoundError` ao rodar | A venv não está ativada, ou faltou o `pip install -r requirements.txt` |
| O Git mostra arquivos inteiros alterados sem ninguém ter mexido | Diferença de fim de linha entre Windows e Linux: crie um `.gitattributes` na raiz com a linha `* text=auto` |

## 3. Como o Django funciona (padrão MVT)

O Django organiza o sistema em três papéis. É o mesmo padrão do projeto que o professor mostrou em aula.

| Papel | Onde fica | O que faz |
|---|---|---|
| Model | models.py | Descreve as tabelas do banco como classes Python |
| Template | templates/ | A tela que o usuário vê (HTML com alguns comandos do Django) |
| View | views.py | Recebe o pedido, busca ou grava dados e escolhe qual template mostrar |

Mais duas peças ajudam: **urls.py** (qual endereço chama qual view) e **forms.py** (valida o que o usuário digita).

### O caminho de um clique

Quando o usuário abre o endereço /caixa/contas/:

1. O navegador envia o pedido (request).
2. O moneyflow/urls.py vê que o endereço começa com caixa/ e passa o pedido para o caixa/urls.py.
3. O caixa/urls.py vê contas/ e chama a view listar_contas.
4. A view pergunta ao model quais contas pertencem ao usuário logado.
5. A view entrega essa lista ao template listar_contas.html.
6. O template vira HTML pronto e volta para o navegador (response).

Quando o usuário envia um formulário (por exemplo, para criar uma conta), entra o forms.py: ele valida os campos antes de qualquer coisa ser gravada.

### Exemplo mínimo (ilustrativo)

É só para mostrar como as peças se encaixam. O código de verdade será escrito nos cards.

```python
# caixa/urls.py
from django.urls import path
from . import views

app_name = 'caixa'
urlpatterns = [
    path('contas/', views.listar_contas, name='listar_contas'),
]
```

```python
# caixa/views.py
from django.contrib.auth.decorators import login_required
from django.shortcuts import render
from .models import ContaBancaria

@login_required
def listar_contas(request):
    contas = ContaBancaria.objects.filter(usuario=request.user)
    return render(request, 'caixa/listar_contas.html', {'contas': contas})
```

```html
<!-- templates/caixa/listar_contas.html -->
{% extends 'base.html' %}
{% block content %}
  <h1>Minhas contas</h1>
  {% for conta in contas %}
    <p>{{ conta.nome }} — R$ {{ conta.saldo_atual }}</p>
  {% empty %}
    <p>Nenhuma conta cadastrada.</p>
  {% endfor %}
{% endblock %}
```

Repare em duas coisas: o @login_required impede que quem não está logado veja a tela, e o filter(usuario=request.user) garante que cada pessoa só veja as próprias contas. Esse filtro vai em toda consulta de dados de usuário.

## 4. Mapa do repositório

```text
Gerenciador_financas/
├── docker-compose.yml
├── README.md
├── docs/
└── src/
    ├── manage.py
    ├── requirements.txt
    ├── .env.example
    ├── moneyflow/
    ├── accounts/
    ├── caixa/
    ├── investimentos/
    ├── patrimonio/
    ├── templates/
    └── static/
```

| Item | O que é | Quando mexer |
|---|---|---|
| docker-compose.yml | Define o container do PostgreSQL | Raramente |
| docs/ | Documentação da disciplina (visão, modelagem, planejamento, API) | Quando uma decisão mudar |
| manage.py | O comando principal do Django (runserver, makemigrations, migrate, startapp) | Nunca editar |
| requirements.txt | Lista das bibliotecas com versões exatas | Depois de cada pip install, com pip freeze |
| .env.example e .env | Variáveis como a senha do banco e a chave secreta. O .env é só local e nunca vai para o Git | Ao criar uma variável nova |
| moneyflow/ | A configuração do projeto (settings.py e urls.py principal) | Pouco |
| accounts/, caixa/, investimentos/, patrimonio/ | Os apps: onde fica quase todo o código | Todo dia |
| templates/ | As telas HTML | Ao criar ou mudar telas |
| static/ | CSS, JavaScript e imagens | Ao mudar o visual |

## 5. O que é um app e o que tem dentro dele

Um app é uma pasta com tudo de um assunto do sistema. Pense assim: o projeto é a casa e cada app é um cômodo. Dividimos por assunto para que cada pessoa trabalhe numa pasta diferente, com menos conflito no Git. O projeto do professor também tem mais de um app (alunos e api).

| Arquivo | Para que serve |
|---|---|
| models.py | As tabelas do banco |
| forms.py | Os formulários e a validação do que o usuário digita |
| views.py | A lógica de cada tela |
| urls.py | As rotas do app. Precisa criar na mão, o startapp não gera |
| admin.py | Registra os models no painel /admin |
| tests.py | Os testes automáticos do app |
| migrations/ | O histórico de mudanças do banco. É gerado pelo Django, mas vai para o Git |
| apps.py | A configuração do app. Não mexe |
| services.py | Regras de negócio com várias tabelas. Só existe onde precisa (caixa, investimentos, patrimonio) |

### Regra de ouro: o que vai em cada lugar

- **View:** recebe o pedido, chama quem resolve e responde. Não faz conta.
- **Model:** os dados e as regras simples da própria tabela.
- **Form:** validação do que entra.
- **services.py:** regra de negócio que envolve várias tabelas, como mexer em saldo.
- **Template:** só apresentação. Nenhuma regra de negócio.

## 6. Os quatro apps

Os campos abaixo seguem o diagrama de classes. A fonte de verdade é o dicionário de dados em docs/modelagem.

### accounts — quem é o usuário

- **Responsabilidade:** cadastro, login e logout.
- **Models:** nenhum. Usamos o User padrão do Django (nome de usuário, e-mail e senha guardada em hash). A tabela se chama auth_user.
- **forms.py:** o formulário de cadastro, baseado no UserCreationForm.
- **views.py:** a view register. Login e logout usam views prontas do Django. Desde o Django 5, o logout exige POST, então o botão Sair fica dentro de um formulário com csrf_token.
- **Templates:** login.html e register.html.
- **Rotas:** /login/, /logout/ e /cadastro/. Quando elas existirem, configure o LOGIN_URL no settings.py.

### caixa — contas, categorias e transações

- **Models:**
  - ContaBancaria: usuario, nome, tipo, saldo_atual.
  - Categoria: usuario, nome, natureza (RECEITA ou DESPESA).
  - Transacao: usuario, conta, categoria, valor, descricao, data.
- **forms.py:** ContaForm, CategoriaForm e TransacaoForm. O TransacaoForm valida que a conta e a categoria escolhidas são do próprio usuário.
- **views.py:** listar, criar, editar e excluir contas e categorias. Listar (com filtros), criar e cancelar transações.
- **services.py:** registrar_transacao, cancelar_transacao, creditar_conta e debitar_conta. Tudo dentro de transaction.atomic.
- **Templates (templates/caixa/):** listagens e formulários de contas, categorias e transações.
- **Rotas:** /caixa/contas/, /caixa/categorias/ e /caixa/transacoes/.
- **Por que as três juntas:** uma transação não existe sem conta e categoria, e o saldo mexe nas três ao mesmo tempo.

### investimentos — ativos e ordens

- **Models:**
  - Ativo: nome, ticker, classe.
  - OrdemInvestimento: usuario, ativo, conta, tipo (COMPRA ou VENDA), quantidade, preco_unitario, data.
- **forms.py:** OrdemForm.
- **Arquivos extras:**
  - cotacoes.py busca preços na Brapi (ações e FIIs) e na CoinGecko (criptomoedas), com a biblioteca requests, tempo limite e um plano B quando a API cair. Instale o requests só quando chegar nesse card.
  - renda_fixa.py tem a função que estima o valor da renda fixa.
- **services.py:** registrar_ordem valida, grava a ordem e chama o service do caixa para debitar a conta (compra) ou creditar (venda). Também calcula a posição do usuário em cada ativo.
- **Templates (templates/investimentos/):** a carteira e o formulário de ordem.
- **Rotas:** /investimentos/...

### patrimonio — o retrato final

- **Models:** HistoricoPatrimonial: usuario, data, total_caixa, total_investido.
- **services.py:** consolidar_patrimonio (soma os saldos das contas com o valor investido) e gerar_snapshot (grava o retrato do dia).
- **Comando:** patrimonio/management/commands/gerar_snapshot.py, que roda uma vez por dia para gravar o histórico.
- **Templates (templates/patrimonio/):** o dashboard e o histórico.
- **Rotas:** /patrimonio/...

## 7. Quem pode chamar quem

O fluxo de dependência é um só: **patrimonio → investimentos → caixa → User**.

1. A seta só vai nesse sentido. O caixa nunca importa nada de investimentos.
2. Um app não altera os models de outro. Ele chama o service. Por exemplo, o investimentos não faz a conta do saldo na mão: ele chama caixa.services.debitar_conta. Assim a regra do saldo existe em um lugar só.
3. Ler dados de outro app é permitido (o patrimonio soma os saldos do caixa, por exemplo).

## 8. templates/ e static/

- **templates/base.html** é o esqueleto de todas as telas (menu, cabeçalho, rodapé, mensagens). As outras telas herdam dele com {% extends 'base.html' %} e preenchem o {% block content %}.
- **templates/nome-do-app/** guarda as telas de cada app.
- **Formulários** sempre levam {% csrf_token %}. Sem ele, o Django bloqueia o envio, e é isso que protege contra formulários forjados.
- **static/css, static/js e static/images** guardam o visual. Para usar, coloque {% load static %} no topo do template e chame com {% static 'css/style.css' %}.
- **JavaScript** é puro e só para interações pontuais (máscara de valor, confirmar exclusão, filtros). Não usamos frameworks de front.

## 9. A pasta moneyflow/ (configuração)

- **settings.py:** os pontos que importam são INSTALLED_APPS (lista dos apps), DATABASES (lê do .env), TEMPLATES com a pasta templates/, STATICFILES_DIRS com a pasta static/ e o idioma e fuso do Brasil.
- **urls.py:** o roteador principal. Ele só distribui os endereços entre os apps:

```python
# moneyflow/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('accounts.urls')),
    path('caixa/', include('caixa.urls')),
    path('investimentos/', include('investimentos.urls')),
    path('patrimonio/', include('patrimonio.urls')),
]
```

- **wsgi.py e asgi.py:** pontos de entrada para produção. Não mexe.

## 10. Onde eu programo cada tarefa

| Quero... | Mexo em... |
|---|---|
| Criar ou alterar uma tabela ou campo | models.py, depois makemigrations e migrate |
| Criar uma tela nova | urls.py + views.py + um template |
| Validar o que o usuário digitou | forms.py |
| Fazer uma regra que mexe no saldo | services.py do caixa |
| Mudar o visual | static/css e os templates |
| Cadastrar ativos ou ver dados rápido | admin.py e o painel /admin |
| Testar uma regra | tests.py do app |
| Usar uma biblioteca nova | pip install, depois pip freeze e commit do requirements.txt |

## 11. Migrations e Git

- Mudou um model? Rode `python manage.py makemigrations` e depois `python manage.py migrate`. **Commite o arquivo de migration junto com o model.**
- Depois de cada `git pull`, rode `python manage.py migrate`.
- Evite duas pessoas mudando os models do mesmo app ao mesmo tempo. Se der conflito de migration, avise o grupo antes de resolver.
- Uma branch por card, PR revisado por outra pessoa e nunca commitar o .env.

## 12. A API REST (marco final)

O README prevê uma API própria para integrações futuras. Ela vai ser um app chamado api, como o do projeto do professor: o serializers.py faz o papel do forms.py, só que para JSON, e os viewsets reaproveitam os services. O djangorestframework só é instalado nesse momento, então a API não trava o resto do projeto.

## 13. Decisões já tomadas e pendentes

**Já decidido:**

- Django com templates, HTML, CSS e JavaScript puros.
- Usuário padrão do Django, com login por sessão.
- PostgreSQL no Docker, Django rodando na máquina de cada pessoa.
- Ordens de investimento debitam ou creditam uma conta bancária.
- Apps divididos por assunto e services.py só onde precisa.

**Ainda em aberto:**

- A compra ou venda de ativo também gera uma transação visível no extrato da conta, ou só altera o saldo?
- Onde guardar o último preço válido das cotações (um campo no Ativo ou uma tabela própria).
- Quem cadastra os ativos (proposta: o administrador, pelo /admin).
- O campo data do histórico patrimonial deve ser do tipo data (um registro por dia).

## 14. Perguntas que o professor pode fazer

**Por que dividir em quatro apps?** Cada app cuida de um assunto, o que deixa o código organizado e permite que cada pessoa trabalhe numa pasta diferente. O projeto de aula também tem mais de um app.

**Por que PostgreSQL e Docker, e não SQLite?** O sistema lida com dinheiro, e o PostgreSQL tem mais rigor com números decimais e restrições. O Docker sobe o banco com um comando e garante que todos usam o mesmo.

**O que é uma migration?** É o histórico de mudanças do banco escrito em arquivos. Quando o model muda, o makemigrations gera o arquivo e o migrate aplica no banco.

**Por que existe um services.py se na aula a lógica ficava na view?** Atualizar o saldo e gravar a transação precisa acontecer junto. O service concentra isso num só lugar, deixa a view enxuta e pode ser reaproveitado pela API e pelo comando diário.

**O que é transaction.atomic?** Um bloco em que, se algo falhar no meio, o banco desfaz tudo. Assim nunca sobra uma transação gravada sem o saldo atualizado.

**Por que o User padrão do Django?** É o que foi ensinado em aula, já traz login, senha em hash e permissões, e atende ao que o sistema precisa.

**Como garantimos que um usuário não veja os dados de outro?** Toda view usa @login_required e toda consulta filtra por usuario=request.user.

**Para que serve o csrf_token nos formulários?** Ele prova que o formulário veio do nosso site, e o Django bloqueia o envio quando ele falta.

**Por que usar um arquivo .env?** Para que senhas e chaves fiquem fora do código e do Git. Cada pessoa tem o seu .env local, e o repositório só leva o modelo, o .env.example.

**Como a API se encaixa no projeto?** Ela é o último marco, num app separado que reaproveita as regras dos services, igual ao app api do projeto de aula.
