# Missão 1 — Hospedagem de Site Estático com Amazon S3

## O problema de negócio

A **Sabor Caseiro** mantém seu site institucional — a página que apresenta a empresa, o catálogo de produtos e informações de contato — no mesmo servidor físico que hospeda o restante da operação. Esse site não tem formulários dinâmicos, não depende de banco de dados e recebe tráfego relativamente baixo e previsível. Ainda assim, ele consome recursos do servidor principal e fica sujeito às mesmas janelas de manutenção e indisponibilidade que o resto da infraestrutura.

A pergunta que este projeto responde é: **"Como hospedar esse site institucional de forma simples, com alta disponibilidade e baixo custo, sem depender do servidor físico e sem gerenciar infraestrutura de servidor?"**

## Por que S3 (e não uma EC2, por exemplo)

Uma instância EC2 resolveria o problema, mas traria de volta o mesmo tipo de responsabilidade que a Sabor Caseiro está tentando reduzir: sistema operacional para manter atualizado, patches de segurança, e uma máquina ligada 24 horas por dia mesmo que o tráfego do site seja pequeno e constante.

O S3 elimina essa camada inteira:

- É um serviço gerenciado — não há sistema operacional para atualizar, corrigir vulnerabilidades ou escalar manualmente.
- O modelo de custo é pay-per-use (armazenamento + requisições), muito mais barato que uma instância EC2 ligada continuamente para uma carga de trabalho tão simples quanto um site institucional.
- Alta durabilidade (11 noves) e disponibilidade nativas do serviço, sem esforço adicional de configuração de redundância.

Para o caso da Sabor Caseiro, isso significa: o site sai do servidor físico, para de competir por recursos com o sistema principal, e passa a custar uma fração do que custava antes.

## O que foi feito

1. Criação de um bucket S3 dedicado exclusivamente à hospedagem do site institucional da Sabor Caseiro.
2. Upload do `index.html` e demais arquivos estáticos (CSS, imagens do catálogo).
3. Habilitação da opção de hospedagem de site estático (Static Website Hosting) no bucket.
4. **Versionamento habilitado** — decisão deliberada para evitar perda de dados em caso de sobrescrita ou exclusão acidental de arquivos durante futuras atualizações do site.
5. Configuração das políticas de acesso público, restritas apenas à leitura dos objetos necessários para exibição do site (sem acesso de escrita ou exclusão pública).

## Versionamento: por que isso importa

Sem versionamento, se alguém da equipe da Sabor Caseiro sobrescrever o `index.html` com uma versão com erro — por exemplo, ao atualizar o catálogo de produtos —, a versão anterior é perdida para sempre. Com versionamento habilitado, cada objeto mantém seu histórico, permitindo reverter para uma versão anterior em segundos. Isso é especialmente relevante para a Sabor Caseiro, cuja equipe de TI é pequena e não tem uma rotina formal de backup antes de cada alteração no site.

## Acesso público: o cuidado que tomei

Configurar acesso público em S3 é uma das causas mais comuns de vazamento de dados na nuvem, quando feito sem cuidado — buckets abertos para leitura *e* escrita, ou expondo por engano dados que não deveriam ser públicos. Neste projeto, o acesso público foi limitado estritamente à leitura dos objetos do site, via política de bucket (Bucket Policy) explícita para a ação `s3:GetObject`. O **Block Public Access** foi ajustado apenas nas configurações estritamente necessárias para permitir a hospedagem estática — não desabilitado por completo a nível de bucket ou de conta.

## Dificuldades encontradas

- Na primeira tentativa de acesso, o site retornava erro 403 mesmo após o upload dos arquivos. O motivo era que o **Block Public Access** ainda estava ativo em todas as suas quatro configurações por padrão — precisei desmarcar especificamente as opções relacionadas a políticas de bucket, mantendo as demais proteções ativas.
- Tive dúvida inicial sobre a diferença entre a URL do endpoint padrão do S3 (`https://bucket.s3.amazonaws.com`) e a URL de hospedagem de site estático (`http://bucket.s3-website-regiao.amazonaws.com`) — são formatos diferentes, e usar o endpoint errado fazia o navegador tentar baixar o `index.html` em vez de renderizá-lo.
- Ao escrever a Bucket Policy manualmente, errei o ARN do recurso na primeira tentativa (esqueci o `/*` no final, que abrange todos os objetos dentro do bucket) e a política simplesmente não tinha efeito nenhum até eu corrigir.

## Evidências

> **Espaço reservado para os prints da execução:**

![Bucket criado com versionamento habilitado](./evidencias/01-bucket-criado.png)

![Configuração de Static Website Hosting](./evidencias/02-versionamento.png)

![Site da Sabor Caseiro no ar, acessado pela URL pública](./evidencias/03-site-no-ar.png)

## O que eu faria além, em um cenário de produção real

Em um cenário de produção real, o próximo passo seria adicionar o CloudFront na frente do bucket, o que traria CDN (menor latência para clientes da Sabor Caseiro em diferentes regiões) e suporte nativo a HTTPS — hoje uma expectativa mínima de segurança para qualquer site institucional voltado ao público. Junto a isso, um domínio customizado via Route 53 (ex: `saborcaseiro.com.br`) substituiria a URL padrão do S3.
