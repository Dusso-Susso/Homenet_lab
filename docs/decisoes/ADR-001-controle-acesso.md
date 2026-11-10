# ADR-001 — Controle de acesso à implantação do HomeDesk

- Data: 2026-11-10
- Status da decisão: aceita
- Status da implementação: planejada
- Status da validação: pendente
- Controle relacionado: AC-001

## Contexto e problema

A conta comum utilizada no laboratório não consegue implantar arquivos
em `/var/www/homedesk` com as permissões atuais.

Precisamos permitir a implantação por usuários autorizados, mantendo
a propriedade administrativa dos diretórios com `root` e evitando
conceder escrita a todos os usuários do sistema.

## Decisão

Utilizar o grupo dedicado `homedesk-deploy` para controlar quem pode
implantar conteúdo do HomeDesk.

| Item | Definição |
| --- | --- |
| Caminho de referência | `/var/www/homedesk` |
| Proprietário dos diretórios | `root` |
| Grupo dos diretórios | `homedesk-deploy` |
| Permissões planejadas para diretórios | `2775` |
| Usuários autorizados | A registrar na implementação |
| Permissões dos arquivos | A definir conforme sua finalidade |

O modo `2775` concede leitura, escrita e travessia ao proprietário e
ao grupo. Os demais usuários recebem leitura e travessia, sem escrita.

O bit setgid faz novos arquivos e subdiretórios herdarem o grupo do
diretório pai. Ele não garante escrita para o grupo nos novos arquivos:
as permissões de criação e a umask também precisam ser consideradas.

O modo `2775` é destinado aos diretórios desta política, não deve ser
aplicado indiscriminadamente aos arquivos comuns.

## Justificativa

Separar as responsabilidades de administração, implantação e execução:

- Administração: manter a propriedade dos diretórios com `root`.
- Implantação: conceder escrita aos integrantes de `homedesk-deploy`.
- Execução: definir uma identidade de serviço com apenas os acessos
  necessários, sem incluí-la automaticamente no grupo de implantação.

A separação completa depende da configuração posterior da identidade
que executará o serviço.

## Consequências e limitações

- A participação em `homedesk-deploy` concede capacidade de alterar
  o conteúdo nos diretórios abrangidos.
- Usuários com escrita e travessia em um diretório podem, em geral,
  remover ou substituir suas entradas, mesmo que não sejam donos
  dos arquivos.
- Os demais usuários locais poderão listar e atravessar os diretórios.
  Esta política não estabelece confidencialidade do conteúdo.
- A escrita compartilhada em novos arquivos depende também de suas
  permissões de criação; setgid, sozinho, não resolve essa necessidade.
- A política não restringe administradores que atuem como `root`.

## Plano de validação

- [ ] Conferir proprietário, grupo e permissões dos diretórios.
- [ ] Confirmar a associação dos usuários autorizados ao grupo.
- [ ] Testar criação de arquivo por um usuário autorizado, sem sudo.
- [ ] Confirmar que o arquivo criado herda o grupo `homedesk-deploy`.
- [ ] Criar um subdiretório e verificar grupo e herança do setgid.
- [ ] Verificar as permissões dos novos arquivos e o efeito da umask.
- [ ] Testar que um usuário fora do grupo não consegue criar arquivos,
      usando uma sessão sem privilégios administrativos.
- [ ] Após definir a identidade do serviço, testar seus acessos.

## Evidências

Validação pendente. Registrar posteriormente os comandos executados,
os resultados observados e a data dos testes.