<h1 align="center">KillTheContext</h1>

<p align="center">
  <strong>Português</strong> · <a href="README.en.md">Read in English</a>
</p>

**Antes de começar: baixe e instale o Orca IDE. Instale também a CLI do Orca caso ela não venha instalada por padrão.** Abra o projeto no Orca e confirme que a CLI está disponível no terminal que executa o Claude Code.

Um prompt para criar um hook do Claude Code que acompanha o tamanho do contexto e prepara a continuidade do trabalho em uma nova sessão no Orca.

## Como usar

1. Tenha o Orca IDE, a CLI do Orca, o Claude Code e o Python instalados.
2. Abra no Orca **o projeto em que você quer aplicar o hook**.
3. Inicie o Claude Code nesse projeto e envie o prompt abaixo.
4. Revise os arquivos gerados e peça ao Claude para demonstrar os testes com limites baixos antes de usar os valores padrão.

Este repositório compartilha o prompt de instalação. O script Python e a configuração são gerados pelo Claude dentro do seu projeto; clonar este repositório não instala nenhum hook.

## O que o hook deve fazer

| Momento | Comportamento solicitado |
| --- | --- |
| Contexto acima de 150.000 tokens | Pedir, uma vez por sessão, um `STATUS.md` com o estado do trabalho e continuar. |
| Contexto acima de 200.000 tokens | Bloquear ferramentas, permitindo apenas leitura, espera e edição do `STATUS.md`. Preservar processos em execução. |
| Evento `Stop` após o limite | Abrir uma nova aba no Orca somente se o `STATUS.md` tiver sido escrito após o limite e não houver processos em segundo plano. Fazer isso uma vez por sessão. |

A medida solicitada é a soma de `input_tokens`, `cache_read_input_tokens` e `cache_creation_input_tokens` da última resposta do assistente no transcript. O hook não reduz o histórico da sessão atual: ele solicita um resumo e prepara uma nova sessão para continuar a partir dele.

## Prompt para enviar ao Claude

Copie o bloco inteiro e envie **dentro do projeto de destino**:

```text
Crie um hook do Claude Code neste projeto que controla o tamanho do contexto. Eu uso o Orca.
Registre o mesmo script Python em .claude/settings.json nos eventos UserPromptSubmit, PreToolUse, PostToolUse e Stop.
O script lê o transcript_path que vem no hook e pega o tamanho do contexto da última resposta do assistente
(input_tokens + cache_read_input_tokens + cache_creation_input_tokens).
- Acima de AVISO (padrão 150000): uma vez por sessão, peça pra escrever um STATUS.md com o que está feito, o que está rodando, o próximo passo exato e as decisões que eu tomei. Depois continua.
- Acima de LIMITE (padrão 200000): negue todas as ferramentas, exceto ler, esperar e editar o STATUS.md. Nunca pare nem mate processos que estejam rodando.
- No evento Stop: se o STATUS.md foi escrito depois do LIMITE e nenhum processo em segundo plano está rodando, abra uma aba nova no Orca com:
  orca terminal create --worktree path:<pasta do projeto> --title "continue" --command "claude --permission-mode auto \"continue a partir do STATUS.md\"" --focus
  (não use orca agent launch, porque ele abre o Claude com --dangerously-skip-permissions). Faça isso uma vez só por sessão.
- AVISO e LIMITE configuráveis por variável de ambiente, uma variável pra desligar a abertura automática e outra pra só registrar num log sem abrir nada. Qualquer erro no script: sai sem fazer nada.
Me mostre como testar com limites baixos antes de usar de verdade.
```

## Teste antes de usar

Como o Claude gera a implementação, os nomes das variáveis e o caminho do log devem ser definidos e documentados por ele. Você pode enviar este complemento:

```text
Antes de ativar o hook, documente os nomes exatos das variáveis de ambiente e o caminho do log. Preserve os hooks e as configurações existentes.
Mostre os comandos adequados ao meu sistema para testar com AVISO=1000 e LIMITE=2000, usando os nomes reais das variáveis implementadas e iniciando uma nova sessão do Claude com elas.
Comece no modo que só registra em log, sem abrir abas. Use um transcript de teste com valores conhecidos para validar os limiares sem precisar gastar tokens de verdade.
Teste: abaixo de AVISO; acima de AVISO com aviso único por sessão; acima de LIMITE com bloqueio das ferramentas e exceções restritas; STATUS.md desatualizado; STATUS.md escrito após o limite; processo em segundo plano ativo; abertura automática desativada; evento Stop repetido; e entrada inválida sem efeitos colaterais.
Não execute nem encerre processos reais para simular esses casos. Não considere a ausência de informação como prova de que não há processos em segundo plano; se não for possível confirmar, não abra uma nova sessão.
Depois, comprove que a abertura acontece uma única vez em uma sessão de teste sem processos em segundo plano, com STATUS.md atualizado e abertura automática habilitada.
Ao terminar, mostre como restaurar AVISO=150000 e LIMITE=200000, sair do modo de log e desativar ou remover o hook se necessário.
```

Os números de teste acima representam os valores desejados, não nomes de variáveis já implementadas. Os testes e o suporte aos eventos precisam ser verificados na versão do Claude Code e do Orca usada no seu projeto. Este repositório não inclui uma implementação executável ou testes de integração já validados.
