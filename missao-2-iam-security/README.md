# Missão 2 — Ambiente Seguro com AWS IAM

## O problema de negócio

A equipe de TI da Sabor Caseiro tem apenas duas pessoas, e até hoje ambas compartilham a mesma credencial de acesso administrativo ao ambiente — uma prática comum em empresas pequenas, mas que traz riscos concretos: não há como saber quem fez qual alteração, uma credencial comprometida dá acesso total ao ambiente, e não existe nenhuma distinção entre quem precisa apenas consultar informações e quem precisa efetivamente administrar recursos.

Com o site institucional já migrado para o S3 na Missão 1, surge a pergunta que toda empresa em crescimento precisa responder antes de escalar sua operação em nuvem: **quem pode acessar o quê?**

Este projeto resolve isso aplicando controle de acesso granular via IAM, eliminando o compartilhamento de credenciais e preparando o terreno para que novos serviços (como o EC2, usado nesta mesma missão) se autentiquem sem depender de chaves fixas.

## O que foi feito

1. **Usuário Admin**: criado para representar a pessoa responsável pela administração geral do ambiente AWS da Sabor Caseiro — substituindo o uso da conta root e da credencial única compartilhada.
2. **Usuário Read-Only**: criado para representar o segundo integrante da equipe, que hoje precisa apenas acompanhar o que está configurado no ambiente (para fins de suporte e auditoria), sem necessidade de alterar nada.
3. **Role para EC2 acessar o S3**: criada uma IAM Role atribuída a uma instância EC2 (que futuramente processará pedidos do catálogo online da Sabor Caseiro), permitindo que ela acesse o bucket S3 da Missão 1 sem usar chaves de acesso fixas (access keys) armazenadas na instância.

## Usuário vs. Grupo vs. Role — a diferença que importa

- **Usuário (User)**: representa uma identidade única e permanente — geralmente uma pessoa, como os dois integrantes da equipe de TI da Sabor Caseiro. Tem credenciais próprias (senha para o console e/ou access keys para linha de comando).
- **Grupo (Group)**: é uma forma de organizar usuários e aplicar permissões em lote. Em vez de configurar permissões usuário por usuário, a permissão é definida uma vez no grupo, e usuários são adicionados ou removidos dele conforme mudam de função — algo que se tornará mais relevante para a Sabor Caseiro à medida que a equipe crescer.
- **Role**: é uma identidade *temporária*, assumida por quem (ou o que) precisa dela — pode ser um usuário, uma aplicação ou, como neste projeto, um serviço AWS (a instância EC2). A grande vantagem é que a Role não tem credenciais fixas: ela gera credenciais temporárias automaticamente, o que elimina o risco de uma chave de acesso vazar e ser usada indefinidamente por quem a encontrar.

Foi exatamente por essa razão que optei por uma **Role** para a EC2 acessar o S3, em vez de gerar uma access key fixa e colocá-la na instância: isso elimina uma credencial permanente que poderia vazar — por exemplo, se a instância fosse comprometida ou se alguém da equipe, por engano, subisse o código com a chave embutida para um repositório público.

## Princípio do Menor Privilégio na prática

O Princípio do Menor Privilégio significa dar a cada identidade **apenas as permissões estritamente necessárias para sua função** — nem mais, nem menos. Neste projeto, isso foi aplicado assim:

- O usuário Read-Only recebeu a policy gerenciada `AmazonS3ReadOnlyAccess`, sem qualquer permissão de escrita, exclusão ou alteração de configurações — suficiente para que esse integrante da equipe acompanhe o ambiente sem risco de alterar algo por engano.
- A Role da EC2 recebeu uma policy customizada, com permissão apenas para as ações específicas necessárias no bucket da Missão 1 (`s3:GetObject` e `s3:PutObject`, restritas ao ARN daquele bucket específico), e não um acesso administrativo total ao S3 ou a outros serviços da conta.
- O usuário Admin, embora tenha permissões amplas via policy `AdministratorAccess`, foi configurado com autenticação multifator (MFA) obrigatória, e passa a ser o único ponto de administração do ambiente — a conta root da Sabor Caseiro deixou de ser usada no dia a dia a partir deste projeto.

## Dificuldades encontradas

- Ao criar a Role para a EC2, esqueci de anexar a **Instance Profile** à instância depois de criar a Role — a permissão existia dentro do IAM, mas a EC2 não conseguia efetivamente usá-la até eu associar corretamente o Instance Profile nas configurações da instância.
- Ao escrever a policy customizada da Role manualmente pelo editor JSON, na primeira versão deixei o campo `Resource` apontando para `"*"` em vez do ARN específico do bucket da Sabor Caseiro — o que na prática dava à Role acesso de leitura e escrita a *qualquer* bucket da conta, não só ao pretendido. Precisei revisar e restringir explicitamente ao ARN correto.
- Tive que testar na prática se a restrição do usuário Read-Only realmente funcionava: tentei, autenticado como esse usuário, excluir um objeto do bucket da Missão 1 e recebi corretamente um erro de acesso negado (`AccessDenied`) — o que confirmou que a policy estava configurada como esperado.

## Evidências

> **Espaço reservado para os prints da execução:**

![Usuários IAM criados: Admin e Read-Only](./evidencias/01-usuarios-iam.png)

![Policy Read-Only aplicada ao segundo usuário](./evidencias/02-policy-readonly.png)

![Role criada com sua Trust Policy, mostrando que apenas o serviço EC2 pode assumi-la](./evidencias/03-role-trust-policy.png)

![Teste prático: usuário Read-Only recebendo AccessDenied ao tentar excluir um objeto](./evidencias/04-teste-acesso-negado.png)

## O que eu faria além, em um ambiente de produção real

O próximo passo natural, à medida que a Sabor Caseiro contratar mais pessoas para a equipe técnica, seria migrar de usuários IAM individuais para o **IAM Identity Center**, centralizando a gestão de identidades e permitindo login único (SSO) em vez de múltiplas credenciais isoladas. Também adicionaria o **IAM Access Analyzer**, que identifica automaticamente permissões concedidas além do necessário — uma forma de auditar continuamente se o Princípio do Menor Privilégio continua sendo respeitado conforme o ambiente cresce.
