# Missão 2 — Controle de Acesso e Segurança com AWS IAM

## O Cenário

Com o site institucional da Sabor Caseiro já migrado para o S3 (Missão 1), a infraestrutura começou a crescer. O problema é que a equipe de TI (composta por duas pessoas) usava a mesma credencial administrativa para tudo. 

Isso gera um risco enorme: falta de rastreabilidade, risco de vazamento com acesso total e ausência de separação de funções. A prioridade desta missão foi organizar a casa e definir exatamente **quem pode acessar o quê**, preparando o ambiente para escalar com segurança.

## O que foi implementado

Para eliminar o compartilhamento de contas e o uso do usuário root no dia a dia, estruturei o acesso da seguinte forma:

* **Usuário Admin:** Criado para a administração do ambiente, exigindo autenticação multifator (MFA) ativada.
* **Usuário Read-Only:** Criado para o segundo membro da equipe, focado em suporte e auditoria. Ele só tem permissão de leitura, garantindo que nada seja alterado ou apagado por acidente.
* **Acesso da aplicação (EC2):** Em vez de criar um usuário e gerar chaves estáticas (*access keys*) para a máquina — o que seria um risco de segurança se a chave vazasse —, criei uma **IAM Role** e anexei à instância EC2. Assim, a aplicação gera credenciais temporárias automaticamente para se comunicar com o S3.

## O Princípio do Menor Privilégio na prática

Toda a configuração foi baseada em dar apenas as permissões estritamente necessárias:

* O usuário de suporte recebeu apenas a policy gerenciada `AmazonS3ReadOnlyAccess`.
* A Role da EC2 recebeu uma policy customizada apenas com `s3:GetObject` e `s3:PutObject`.
* A policy da EC2 foi travada especificamente no ARN do bucket da Missão 1, impedindo que ela acesse qualquer outro bucket da conta.

## Desafios no meio do caminho

* **Esquecimento do Instance Profile:** Criei a Role para a EC2 no IAM, mas a máquina continuava sem acesso. Demorei um pouco para perceber que eu precisava atrelar o *Instance Profile* diretamente nas configurações da instância para a permissão funcionar de fato.
* **Atenção ao JSON da Policy:** Na primeira versão da policy da Role, deixei o campo `Resource` como `"*"` por força do hábito. Quando fui revisar, vi que a EC2 teria acesso a *qualquer* bucket da conta. Voltei e corrigi apontando para o ARN específico do bucket da Sabor Caseiro.
* **Validação prática:** Fiz questão de logar com o usuário Read-Only e tentar deletar um arquivo do S3 para testar a segurança. O erro `AccessDenied` apareceu como esperado, validando a restrição da policy.

## Evidências

> **Espaço reservado para os prints da execução:**

![Usuários IAM criados: Admin e Read-Only](./evidencias/01-usuarios-iam.png)
![Policy Read-Only aplicada ao segundo usuário](./evidencias/02-policy-readonly.png)
![Role criada com sua Trust Policy, limitando o AssumeRole ao serviço EC2](./evidencias/03-role-trust-policy.png)
![Teste prático: usuário Read-Only recebendo AccessDenied ao tentar excluir um objeto](./evidencias/04-teste-acesso-negado.png)

## Próximos passos (Visão de Produção)

Pensando no crescimento da equipe técnica da Sabor Caseiro, o próximo passo ideal seria abandonar a gestão de usuários isolados no IAM e migrar para o **IAM Identity Center (SSO)** para centralizar as identidades. Também valeria a pena ativar o **IAM Access Analyzer** para monitorar continuamente se não estamos deixando permissões excessivas passarem despercebidas com o tempo.
