# Como publicar este repositório no GitHub

## 1. Antes de publicar (importante)

Os textos já estão 100% escritos, mas com uma narrativa plausível que **eu inventei** para preencher a estrutura (nomes de policies, erros específicos, detalhes técnicos). Antes de subir:

- [ ] Execute de verdade as três missões na sua conta AWS
- [ ] Compare o que aconteceu na prática com o texto de cada README — ajuste nomes de policies, regiões, mensagens de erro e as "Dificuldades encontradas" para refletir o que você realmente viveu
- [ ] Tire os prints e coloque nas pastas `evidencias/` de cada missão, com os nomes de arquivo já referenciados nos READMEs (ex: `01-bucket-criado.png`)
- [ ] Troque o `#` do LinkedIn/e-mail no README principal pelo seu link real
- [ ] (Opcional) Ajuste o diagrama Mermaid da Missão 3 se sua arquitetura tiver detalhes diferentes

Isso importa porque um entrevistador pode perguntar sobre qualquer detalhe do texto — a credibilidade do portfólio depende de bater com sua experiência real.

## 2. Criar o repositório no GitHub

1. Acesse [github.com/new](https://github.com/new)
2. Nome sugerido: `portfolio-cloud-aws`
3. Deixe **Public** (recrutador precisa conseguir ver)
4. **Não** marque "Add a README" (você já tem um) — deixe o repositório vazio
5. Clique em **Create repository**

## 3. Subir o projeto (comandos no PuTTY/terminal, dentro da pasta extraída)

Extraia o zip, entre na pasta `portfolio-cloud` e rode, em ordem:

```bash
cd portfolio-cloud
git init
git add .
git commit -m "Primeira versão do portfólio: S3, IAM e plano de migração CAF/Well-Architected"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/portfolio-cloud-aws.git
git push -u origin main
```

> Troque `SEU-USUARIO` pelo seu usuário do GitHub. Se pedir login, use um Personal Access Token no lugar da senha (o GitHub não aceita mais senha comum via linha de comando).

## 4. Depois de publicar

- Confira no navegador se as imagens aparecem corretamente nos três READMEs
- Confira se o diagrama Mermaid da Missão 3 renderizou (o GitHub faz isso automaticamente em blocos ```mermaid)
- Adicione o link do repositório no seu LinkedIn, na seção "Destaques" ou em um post
- Fixe (pin) o repositório no seu perfil do GitHub para aparecer em destaque
