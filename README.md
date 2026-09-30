# Teacher Tiago — Gestão de alunos

Plataforma web para gerenciar alunos, agenda, aulas, evolução e financeiro das aulas de inglês do Teacher Tiago.

Feita em **HTML, CSS e JavaScript puros**, sem frameworks e sem etapa de build. Funciona em qualquer hospedagem de site estático (GitHub Pages, Netlify, Vercel etc.) e também abrindo o `index.html` direto no navegador.

---

## Estrutura

```
teacher-tiago-gestao/
├── index.html              Estrutura da página e ordem dos scripts
├── css/
│   └── style.css           Todo o visual (cores da marca em :root, no topo)
├── assets/
│   └── favicon.svg         Ícone da aba do navegador
├── js/
│   ├── config.js           ← Configuração do Firebase (vazio = modo local)
│   ├── main.js             Inicialização: escolhe o banco e abre a plataforma
│   ├── core/
│   │   ├── helpers.js      Datas, formatação de valores, constantes, ícones
│   │   ├── state.js        Estado global e acesso ao banco
│   │   ├── lessons.js      Motor das aulas recorrentes
│   │   └── payments.js     Motor das mensalidades (pós-pagas)
│   ├── views/              Uma tela por arquivo
│   │   ├── dashboard.js
│   │   ├── alunos.js       Lista, busca e filtros
│   │   ├── aluno.js        Perfil individual (abas)
│   │   ├── calendario.js   Semana, dia e mês
│   │   ├── financeiro.js
│   │   └── configuracoes.js
│   ├── ui/
│   │   ├── render.js       Rotas (#/dashboard, #/alunos…) e desenho da tela
│   │   ├── components.js   Toast e pequenos componentes
│   │   ├── layer.js        Modais e gaveta lateral
│   │   ├── forms.js        Formulários
│   │   └── events.js       Cliques e envio de formulários
│   └── data/
│       ├── local-db.js     Banco no navegador (localStorage)
│       ├── firebase.js     Login + banco na nuvem (Firestore)
│       ├── sync.js         Atualização dos dados em tempo real
│       └── backup.js       Exportar e importar backup (.json)
└── firestore.rules         Regras de segurança para colar no Firebase
```

Os scripts são carregados em ordem pelo `index.html` (sem módulos ES), para que o projeto funcione até abrindo o arquivo com duplo clique. Se criar um arquivo `.js` novo, adicione a tag `<script>` dele no `index.html`, antes de `js/main.js`.

---

## Onde guardar os dados: dois modos

A plataforma escolhe sozinha, de acordo com o `js/config.js`.

**Modo local (padrão, bloco `firebase` vazio)**
- Não precisa de nenhuma configuração nem de login.
- Os dados ficam salvos no navegador (localStorage) do aparelho em que foram cadastrados.
- Não sincroniza entre aparelhos. Limpar os dados do navegador apaga tudo, por isso use **Configurações → Baixar backup dos dados** com frequência.
- Ideal para testar ou para uso em um único computador.

**Modo nuvem (bloco `firebase` preenchido)**
- Tela de login com e-mail e senha.
- Dados salvos no Firestore (Google), acessíveis de qualquer aparelho, em tempo real.
- Só os e-mails liberados em `firestore.rules` conseguem ver e editar.

Para passar do modo local para a nuvem sem perder nada: baixe o backup no modo local, configure o Firebase, entre e use **Configurações → Importar backup**.

---

## Configurar o Firebase (modo nuvem)

1. Acesse https://console.firebase.google.com → **Adicionar projeto** (o Google Analytics pode ficar desativado).
2. **Authentication** → **Vamos começar** → **E-mail/senha** → ativar. Na aba **Usuários**, clique em **Adicionar usuário** para cada pessoa que vai usar a plataforma.
3. **Firestore Database** → **Criar banco de dados** → local `southamerica-east1 (São Paulo)` → **modo de produção**.
4. Na aba **Regras** do Firestore, cole o conteúdo de `firestore.rules`, troque os e-mails e clique em **Publicar**.
5. Engrenagem → **Configurações do projeto** → **Seus apps** → ícone `</>` → registre o app e copie os valores do `firebaseConfig` para o `js/config.js`.
6. Em **Authentication → Configurações → Domínios autorizados**, adicione o domínio onde o site ficará (ex.: `oteachertiago.github.io`).

Os valores do `firebaseConfig` não são senhas e podem ficar públicos no GitHub. Quem protege os dados são as regras do passo 4.

O plano gratuito do Firebase (Spark) cobre com folga o uso de um professor particular.

---

## Publicar no GitHub Pages

1. Crie um repositório e envie **todos os arquivos e pastas** deste projeto (o `index.html` precisa ficar na raiz).
2. **Settings → Pages** → Branch `main`, pasta `/ (root)` → **Save**.
3. Em 1–2 minutos o site fica em `https://SEU-USUARIO.github.io/NOME-DO-REPOSITORIO/`.

Todos os caminhos são relativos, então o projeto funciona em qualquer subpasta ou domínio.

---

## Modelo de dados

| Coleção        | Documento                    | Conteúdo principal |
|----------------|------------------------------|--------------------|
| `alunos`       | id automático                | nome, contato, nível, `agenda` (dia da semana → horário), `aulaAtual`, valor mensal, dia de vencimento, evolução, histórico de nível |
| `aulas`        | `idAluno_AAAA-MM-DD`         | só aulas que saíram do padrão: realizadas, remarcadas, canceladas ou avulsas |
| `pagamentos`   | `idAluno_AAAA-MM`            | valor, vencimento, status, data e forma de pagamento |
| `config`       | `geral`                      | nome no painel, horário da agenda, valores dos planos, vencimento padrão |

As aulas recorrentes **não** são gravadas uma a uma: o calendário as gera a partir da `agenda` de cada aluno (`js/core/lessons.js`). Uma aula vira documento só quando algo acontece com ela.

Cobrança pós-paga: as aulas de um mês são cobradas no dia de vencimento do mês seguinte (`js/core/payments.js`).

---

## Editar o visual

- **Cores da marca:** variáveis no topo do `css/style.css` (`--blue`, `--red`, `--yel`…).
- **Fonte:** Montserrat, carregada do Google Fonts no `index.html`.
- **Textos e estrutura de cada tela:** arquivo correspondente em `js/views/`.
- **Menu lateral e barra inferior do celular:** função `drawNav()` em `js/ui/render.js`.
