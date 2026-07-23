# Portfólio Cloud — AWS

Este repositório reúne três projetos práticos de infraestrutura em nuvem, desenvolvidos na minha conta pessoal da AWS. O objetivo não é apenas demonstrar a execução de serviços, mas apresentar as decisões técnicas por trás de cada solução, explicando o problema que ela resolve, as alternativas avaliadas e os desafios encontrados durante a implementação.

Os três projetos fazem parte de um único cenário: a migração fictícia da Sabor Caseiro, um e-commerce de médio porte que atualmente opera em um servidor físico local e precisa modernizar sua infraestrutura para reduzir custos de manutenção, aumentar a disponibilidade e garantir a escalabilidade do ambiente.

## Projetos

| # | Projeto | Foco | Serviços AWS | Status |
|---|---------|------|---------------|--------|
| 1 | [Hospedagem de site estático](./missao-1-s3-static-site) | Armazenamento e custo | S3 | ✅ Concluído |
| 2 | [Ambiente seguro com IAM](./missao-2-iam-security) | Segurança e acesso | IAM, EC2, S3 | ✅ Concluído |
| 3 | [Plano de migração para a nuvem](./missao-3-caf-migration) | Arquitetura e estratégia | CAF, Well-Architected Framework | ✅ Concluído |

## O fio condutor: a Sabor Caseiro

Para tornar os três projetos mais próximos de um cenário real de mercado, e não apenas exercícios isolados, todos giram em torno da mesma empresa fictícia:

> **Sabor Caseiro** é um e-commerce de médio porte que vende produtos alimentícios artesanais. Hoje, o site institucional e o catálogo de produtos rodam em um único servidor físico, alugado em um data center local, sem redundância. A equipe de TI é pequena (duas pessoas) e todos compartilham as mesmas credenciais de acesso ao servidor — uma prática que funcionou enquanto a empresa era pequena, mas que hoje representa um risco real de segurança e de continuidade do negócio.

Cada missão resolve uma parte concreta desse problema, em ordem crescente de maturidade:

1. A **Missão 1** tira o site institucional do servidor físico e o hospeda de forma barata e escalável no S3 — o primeiro passo, e o mais simples, de tirar peso do servidor local.
2. A **Missão 2** resolve o problema mais urgente de segurança: elimina o compartilhamento de credenciais, criando acessos individuais e uma Role para que os próprios serviços da AWS se comuniquem sem depender de chaves fixas.
3. A **Missão 3** dá um passo atrás da execução pontual e apresenta à liderança da Sabor Caseiro um plano estruturado de migração completa do servidor físico restante, usando frameworks oficiais da AWS em vez de uma decisão no improviso.

Essa progressão — execução pontual → segurança → estratégia — reflete o mesmo raciocínio que uma empresa real segue ao amadurecer sua operação em nuvem.

## Stack e ferramentas

- **AWS**: S3, IAM, EC2
- **Frameworks**: AWS Well-Architected Framework, AWS Cloud Adoption Framework (CAF)
- **Diagramação**: Draw.io
- **Documentação**: Markdown

## Como navegar

Cada pasta de missão tem seu próprio README com:
- O problema de negócio que motivou o projeto
- As decisões técnicas tomadas (e o porquê)
- Dificuldades enfrentadas e como foram resolvidas
- Prints e evidências da execução
