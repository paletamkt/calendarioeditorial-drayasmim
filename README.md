# Calendário Editorial — Dra. Yasmim Meneses

Projeto Firebase: `paleta-yasmim`
App: `index.html` (página única, sem build, sem dependências de servidor)

## O que mudou (jun/2026)

O app parou de funcionar periodicamente porque as regras de segurança do Firestore
estavam em **modo de teste**, que expira automaticamente 30 dias depois de criadas
(elas tinham expirado em 26/06/2026). Quando a regra expira, toda leitura ao banco
passa a ser negada — e o código do app, em vez de avisar o erro, caía silenciosamente
em uma lista de posts de exemplo (`DEFAULT_POSTS`, hardcoded no HTML). Por isso parecia
que os dados tinham "zerado", quando na verdade continuavam intactos no Firestore.

Correção aplicada:
- Removida a senha de admin em texto puro (`ADMIN_PASSWORD`) que ficava visível no
  código-fonte da página.
- Adicionado login real via **Firebase Authentication** (e-mail/senha).
- Atualizadas as regras do Firestore para **modo produção sem expiração**: leitura
  liberada para qualquer um (o calendário é público por design), escrita liberada
  só para usuários autenticados.

## Configuração necessária no Firebase Console

### 1. Authentication
`Build → Authentication → Sign-in method` → ativar provedor **E-mail/senha**.

`Authentication → Users → Add user` → criar a conta da equipe (ex:
`equipe@paletamkt.com` + senha forte). Essa é a conta usada por toda a equipe
Paleta Mkt para entrar em "Modo edição" no app. Guardar a senha em gerenciador
de senhas — não em texto no código.

> Se no futuro for necessário diferenciar quem edita (equipe) de quem só
> comenta (Dra. Yasmim), criar contas separadas e ajustar as regras do
> Firestore por papel. Hoje o modelo é uma conta única compartilhada pela
> equipe.

### 2. Firestore → Regras
Substituir o conteúdo por:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

Clicar em **Publicar** depois de colar — só editar o texto no editor não aplica
a regra, é preciso publicar.

## Estrutura de dados (Firestore)

Coleção `posts`, um documento por post:

| campo     | tipo   | observação                                  |
|-----------|--------|----------------------------------------------|
| `type`    | string | `reel` \| `carousel` \| `static`              |
| `date`    | string | formato `YYYY-MM-DD`                          |
| `title`   | string | título / gancho                               |
| `caption` | string | legenda completa                              |
| `image`   | string | base64 (upload) ou URL externa, pode ser vazio |

Imagens enviadas por upload são redimensionadas no navegador via `<canvas>`
para 480×600px com fundo escuro (`#1E1A16`) antes de salvar como base64.

## Login no app

Botão de cadeado no header → modal pede e-mail e senha da conta criada no
Firebase Authentication. Login fica salvo no navegador (sessão padrão do
Firebase Auth), não precisa logar de novo a cada visita no mesmo dispositivo.
Botão "Sair" (mesmo cadeado, já desbloqueado) encerra a sessão.

## Pendências / próximos passos

- [ ] Confirmar conta de e-mail/senha criada em Authentication
- [ ] Publicar a nova regra do Firestore
- [ ] Subir este `index.html` no lugar do arquivo atual hospedado
- [ ] Testar: logar, criar um post de teste, excluir o post de teste
- [ ] Avisar Larissa e Nicholas sobre a nova forma de login (sem mais senha
      fixa "paleta2026")
