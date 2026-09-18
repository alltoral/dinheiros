# Dinheiros

App pessoal de controle financeiro focado em **previsibilidade**: lance suas entradas e saídas (inclusive as fixas/recorrentes) e veja o saldo projetado no futuro.

Desenvolvido por **ALLTORAL**.

## Stack

- HTML puro + React via CDN (sem build, sem Node, sem `npm install`)
- Firebase (Auth com Google + Firestore) para sincronizar os dados entre dispositivos
- PWA: instalável na tela de início do celular, funciona em tela cheia como um app nativo

## Estrutura

```
index.html              → app inteiro (interface + lógica)
sw.js                    → service worker (cache pra abrir offline)
manifest.json            → configuração do PWA (nome, ícone, cores)
icon-192.png              → ícone do app (192x192)
icon-512.png              → ícone do app (512x512)
apple-touch-icon.png      → ícone para iPhone/iPad
larot-avatar.png          → personagem do Larot (mic acordado)
larot-avatar-muted.png    → personagem do Larot (mic parado/erro)
```

## Rodando localmente

Não precisa de instalação. Basta abrir o `index.html` num navegador, ou servir a pasta com qualquer servidor estático:

```bash
python3 -m http.server 8000
```

e acessar `http://localhost:8000`.

## Deploy

O projeto é 100% estático, então funciona em qualquer host de arquivos estáticos:

- **Netlify** (recomendado): conecte este repositório em [app.netlify.com](https://app.netlify.com) → "Add new site" → "Import an existing project" → escolha este repo no GitHub. Todo `git push` na branch principal atualiza o site automaticamente.
- **GitHub Pages**: em Settings → Pages, escolha a branch `main` e a pasta raiz.
- **Vercel**: importe o repositório em vercel.com, sem configuração adicional necessária.

## Configuração do Firebase

O app usa Firebase Auth (login com Google) e Firestore (banco de dados) para sincronizar os lançamentos entre dispositivos. As credenciais em `index.html` (`firebaseConfig`) são públicas por natureza — quem protege os dados são as regras do Firestore, configuradas para que cada usuário só acesse os próprios dados:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /alltoral-fin/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

Se for clonar este projeto para outro uso, troque o `firebaseConfig` pelo de um novo projeto Firebase.

## Larot (assistente de voz)

O personagem no canto inferior direito (logo acima do botão "+" rosa) é o **Larot** — toque nele uma vez pra falar, e de novo quando terminar. Só aparece depois de fazer login com Google.

- **100% local e gratuito** — reconhecimento de padrões em JavaScript (Web Speech API), sem IA nem API paga.
- **Sem resposta falada**, exceto nas consultas de saldo (abaixo), que ele fala em voz alta.
- Valores em dinheiro aceitam vírgula como decimal (ex: "2,37"), igual o resto do app.

**Comandos:**

- **Entrada/saída avulsa** — "entrada de 2,37 pix da Ana" / "saída de 25,87 aluguel". Lança hoje; a descrição é opcional (usa "Entrada"/"Saída" se não falar nada).
- **Parcelamento** — "parcelamento de 250 por 4 meses notebook" — uma parcela por mês, valor igual em todas (mesma lógica do formulário).
- **Lançamento fixo/recorrente** — "saída recorrente de 330 reais todo dia 5, terapia" (ou "entrada recorrente de..."). Repete todo mês no mesmo dia, por 12 meses por padrão (fale "...por 6 meses" pra mudar).
- **Saldo numa data** — "quanto vou ter amanhã" / "quanto vou ter dia 20" / "quanto vou ter no mês que vem". Sem data, é o saldo de hoje. Ele sempre fala se o saldo é **positivo** ou **negativo**, não só o número.
- **Quando volta a ficar positivo** — "quando meu saldo volta a ser positivo?" / "vai ficar negativo até quando?" — acha a próxima data em que o saldo projetado deixa de ser negativo.
- **Quanto falta pra sair do vermelho** — "quanto eu preciso pra sair do vermelho" / "quanto falta pra ficar positivo no mês que vem" — fala quanto a mais (além do que já está lançado) você precisaria pra zerar o saldo naquela data.

Tem um botão 🐞 no canto inferior esquerdo que abre um painel de depuração, mostrando em tempo real o que o Larot está ouvindo/processando — útil pra diagnosticar algo que não funcionou como esperado no celular.
