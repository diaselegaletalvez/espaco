# Espaço

App de Mac que varre o disco, mostra onde os gigabytes estão e limpa o que dá — separando o que se regenera sozinho do que você deveria pensar duas vezes antes de apagar.

Nativo em SwiftUI, sem sandbox, sem assinatura.

> Nome provisório. O definitivo sai quando o app estiver pronto.

## O que ele faz

| Tela | Responde |
|---|---|
| **Revisão** | "O que dá pra limpar agora?" — cinco varreduras, uma lista, marca e executa |
| **Mapa** | "Onde estão os gigabytes?" — treemap navegável do disco inteiro |
| **Apps** | "Quanto esse app realmente ocupa?" — pacote + resíduos espalhados pelo Library |
| **Esquecidos** | "O que eu baixei e nunca mais abri?" — arquivos grandes e instaladores usados |
| **Projetos** | "Que projeto parado está segurando espaço?" — cruza data do último toque com peso das dependências |
| **Saúde** | "Meu Mac está bem?" — memória, CPU, bateria com ciclos e capacidade real |
| **Relatórios** | "O que mudou desde ontem?" — histórico diário em markdown |
| **Automação** | "Como faço isso sozinho?" — agendamento e o manual de comandos |

## Como ele decide o que apagar

Três níveis, sempre visíveis antes de você confirmar:

- **Risco zero** — cache, DerivedData, símbolos de debug. Se regenera sozinho. É apagado direto, e o espaço volta na hora.
- **Risco médio** — `node_modules`, `Pods`, builds. Volta com um `npm install`. Vai pra **Lixeira**.
- **Risco alto** — Archives do Xcode (dSYMs de builds publicados), backups locais de iPhone. Vai pra **Lixeira**, com aviso explícito.

Nada é apagado de forma irrecuperável sem que a categoria diga, em português, o que você perde.

## O manual

Todos os comandos que o app executa estão documentados em **[MANUAL.md](MANUAL.md)** — dez blocos numerados, do mais seguro ao que exige pensar duas vezes, cada um dizendo o que você perde.

Dentro do app eles ficam na aba **Automação** e no botão de livro da sidebar (⌘⇧M), com **Rodar agora** para os que não pedem senha e **Abrir no Terminal** para os que pedem.

## Instalação

Baixe o `.dmg` mais recente em [Releases](../../releases/latest). Se quiser só as automações sem interface, há dois `.pkg`: `riskzero` e `riskmedio`, arraste pra Applications e abra.

Na primeira execução o macOS vai pedir:

- **Notificações** — pro aviso de fim de limpeza
- **Acesso total ao disco** *(recomendado)* — sem ele, o app não consegue medir o `Library` de outros apps e os totais ficam menores que a realidade

Ajustes → Privacidade e Segurança → Acesso total ao disco → adicione o Espaço.

## Onde está o quê

| Repositório | Conteúdo |
|---|---|
| [`espaco`](https://github.com/diaselegaletalvez/espaco) | este site, o manual e os **releases** (`.dmg` e `.pkg`) |
| [`espaco-mac`](https://github.com/diaselegaletalvez/espaco-mac) | o código do app de Mac (`Espaco.xcodeproj`) |

## Compilando

Requer Xcode 16+ e macOS 14+.

```bash
git clone https://github.com/diaselegaletalvez/espaco-mac.git
cd espaco-mac
open Espaco.xcodeproj
```

O target precisa estar **sem App Sandbox** — ler `~/Library` e `/Library` exige isso, e é o motivo de o app ser distribuído fora da App Store.

Pra regerar o ícone:

```bash
swift gerar-icone.swift
```

## Distribuindo

O `distribuir.sh` faz tudo: compila em Release, assina com Developer ID, monta o `.dmg`, manda pra Apple notarizar e grampeia o selo.

```bash
./distribuir.sh
```

Pré-requisitos, uma vez só:

1. Um certificado **Developer ID Application** no chaveiro — Xcode → Settings → Accounts → Manage Certificates → **+**
2. As credenciais de notarização guardadas, com uma [senha de app](https://appleid.apple.com):

```bash
xcrun notarytool store-credentials "espaco-notarizacao" \
  --apple-id "seu@email.com" \
  --team-id "SEUTEAMID" \
  --password "xxxx-xxxx-xxxx-xxxx"
```

Pra testar o build sem esperar a Apple: `./distribuir.sh --sem-notar`. O `.dmg` sai funcionando na sua máquina, mas o Gatekeeper bloqueia em qualquer outra.

## O relatório diário

Um script em `~/bin/mac-report.sh` roda todo dia via `launchd`, mede o disco, limpa o que é risco zero e escreve um markdown em `~/Relatorios`. A aba **Automação** do app instala, agenda e desliga isso pela interface — mas o script funciona sozinho, sem o app.

```
~/Relatorios/
├── mac-2026-09-06.md      relatório do dia
├── ultimo.md              atalho pro mais recente
├── historico.json         números crus, pro gráfico de 30 dias
└── logs/
```

## Estrutura

```
Espaco/
├── Models.swift           tipos base, catálogo de alvos, formatação
├── Scanner.swift          medição de disco e pastas (statfs + getattrlist)
├── Tree.swift             árvore de tamanhos pro mapa
├── Treemap.swift          layout squarified
├── Cleaner.swift          execução das limpezas
├── Uninstaller.swift      apps e seus resíduos
├── BigFiles.swift         arquivos grandes esquecidos
├── Duplicates.swift       duplicados por SHA-256 em três estágios
├── Projects.swift         projetos dormentes
├── Health.swift           memória, CPU, bateria via IOKit e sysctl
├── SystemInfo.swift       identificação da máquina
├── Automation.swift       launchd, script diário e manual de comandos
├── History.swift          leitura dos relatórios
├── LayoutGuard.swift      detector de texto cortado na interface
├── Theme.swift            tokens visuais
└── ...Views
```

## Licença

MIT. Veja [LICENSE](LICENSE).

---

Feito pela **Dias Inc.** — um estúdio, vários mundos.
