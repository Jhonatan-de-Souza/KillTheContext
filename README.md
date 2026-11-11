<h1 align="center">KillTheContext</h1>

<p align="center">
  <strong>Português</strong> · <a href="README.en.md">Read in English</a>
</p>

**Antes de começar: baixe e instale o [Orca IDE](https://www.onorca.dev/download). Instale também a CLI do Orca caso ela não venha instalada por padrão.** Abra o projeto no Orca e confirme que a CLI está disponível no terminal que executa o Claude Code.

Um prompt para criar um hook do Claude Code que acompanha o tamanho do contexto e prepara a continuidade do trabalho em uma nova sessão no Orca.

## Como usar

1. Tenha o [Orca IDE](https://www.onorca.dev/download), a CLI do Orca, o Claude Code e o [Python](https://www.python.org/downloads/) instalados.
2. Abra no Orca **o projeto em que você quer aplicar o hook**.
3. Inicie o Claude Code nesse projeto e envie o prompt abaixo.
4. Revise os arquivos gerados e peça ao Claude para demonstrar os testes com limites baixos antes de usar os valores padrão.

Este repositório compartilha o prompt de instalação. O script Python e a configuração são gerados pelo Claude dentro do seu projeto; clonar este repositório não instala nenhum hook.

## O que o hook deve fazer

| Momento | Comportamento solicitado |
| --- | --- |
| Contexto acima de 350.000 tokens (`AVISO`) | Pedir, uma vez por sessão, um `STATUS.md` com o estado do trabalho e continuar. |
| Contexto acima de 400.000 tokens (`LIMITE`) | Bloquear ferramentas, permitindo apenas leitura, espera e edição do `STATUS.md`. Preservar processos em execução. |
| Evento `Stop` após o limite | Abrir uma nova aba no Orca somente se o `STATUS.md` tiver sido escrito após o limite e não houver processos em segundo plano. Fazer isso uma vez por sessão. |

A medida solicitada é a soma de `input_tokens`, `cache_read_input_tokens` e `cache_creation_input_tokens` da última resposta do assistente no transcript. O hook não reduz o histórico da sessão atual: ele solicita um resumo e prepara uma nova sessão para continuar a partir dele.

## Como mudar os limites

Os padrões são **350.000 tokens** para o `AVISO` (pedido de checkpoint) e **400.000 tokens** para o `LIMITE` (ferramentas bloqueadas). Para mudar o comportamento do hook, edite esses dois números no prompt abaixo, nas duas linhas que dizem `padrão 350000` e `padrão 400000`, antes de enviá-lo ao Claude:

| Valor | Padrão | O que controla |
| --- | --- | --- |
| `AVISO` | `350000` | Quando o Claude é instruído a escrever o `STATUS.md` e continuar. Reduza para fazer o checkpoint mais cedo. |
| `LIMITE` | `400000` | Quando as ferramentas são bloqueadas e o hook prepara a nova sessão. Mantenha acima do `AVISO`. |

Os dois também podem ser sobrescritos depois por variáveis de ambiente, sem editar o script. O Claude define os nomes exatos das variáveis, então peça que ele os documente.

## Prompt para enviar ao Claude

Copie o bloco inteiro e envie **dentro do projeto de destino**:

```text
Crie um hook do Claude Code neste projeto que controla o tamanho do contexto. Eu uso o Orca.
Registre o mesmo script Python em .claude/settings.json nos eventos UserPromptSubmit, PreToolUse, PostToolUse e Stop.
O script lê o transcript_path que vem no hook e pega o tamanho do contexto da última resposta do assistente
(input_tokens + cache_read_input_tokens + cache_creation_input_tokens).
- Acima de AVISO (padrão 350000): uma vez por sessão, peça pra escrever um STATUS.md com o que está feito, o que está rodando, o próximo passo exato e as decisões que eu tomei. Depois continua.
- Acima de LIMITE (padrão 400000): negue todas as ferramentas, exceto ler, esperar e editar o STATUS.md. Nunca pare nem mate processos que estejam rodando.
- No evento Stop: se o STATUS.md foi escrito depois do LIMITE e nenhum processo em segundo plano está rodando, abra uma aba nova no Orca com:
  orca terminal create --worktree path:<pasta do projeto> --title "continue" --command "claude --permission-mode auto \"continue a partir do STATUS.md\"" --focus
  (não use orca agent launch, porque ele abre o Claude com --dangerously-skip-permissions). Faça isso uma vez só por sessão.
- AVISO e LIMITE configuráveis por variável de ambiente, uma variável pra desligar a abertura automática e outra pra só registrar num log sem abrir nada. Qualquer erro no script: sai sem fazer nada.
Me mostre como testar com limites baixos antes de usar de verdade.
```
