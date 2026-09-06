# Manual de limpeza do Mac

Os comandos que o Espaço executa por botão, aqui em ordem, pra rodar à mão.

Vieram de uma faxina real: um MacBook Air de 228 GB que estava com **8,7 GB livres** e terminou com **101 GB**. Cada bloco diz o que você perde antes de você rodar.

**Rode na ordem.** O bloco 0 é obrigatório e vale por janela do terminal. O bloco 9 não apaga nada — é diagnóstico.

---

## 0 · Preparar o terminal

Desliga os prompts `sure you want to delete` do zsh e guarda a senha do sudo pros blocos seguintes.

```bash
setopt rm_star_silent
sudo -v
```

---

## 1 · Resto do Xcode — *risco zero*

Caches de simulador que o sistema guarda fora da sua pasta, e simuladores órfãos de versões antigas do Xcode.

```bash
sudo rm -rf /Library/Developer/CoreSimulator/Caches
xcrun simctl delete unavailable
df -h /System/Volumes/Data | tail -1
```

**Ganho típico:** 3–11 GB

---

## 2 · Runtimes tvOS, watchOS e xrOS — *risco zero*

Se você não faz app pra Apple TV, Watch ou Vision Pro, esses runtimes só ocupam espaço. O iOS fica intacto. O `erase all` demora 1 a 2 minutos.

```bash
for id in $(xcrun simctl runtime list | grep -E '^(tvOS|watchOS|xrOS)' | sed -E 's/.* - ([A-F0-9-]{36}) .*/\1/'); do
  xcrun simctl runtime delete "$id"
done
xcrun simctl runtime delete unusable
xcrun simctl erase all
df -h /System/Volumes/Data | tail -1
```

**Ganho típico:** 15–45 GB — costuma ser o maior de todos

---

## 3 · Caches de dev — *risco zero*

CocoaPods, npm, Homebrew, Expo, Electron, Yarn, pnpm, gradle. Tudo volta com um install. Só o próximo build fica mais lento.

```bash
rm -rf ~/Library/Caches/CocoaPods
rm -rf ~/Library/Caches/com.microsoft.VSCode.ShipIt
rm -rf ~/Library/Caches/dotslash
rm -rf ~/Library/Caches/ReactNative
rm -rf ~/Library/Caches/electron
rm -rf ~/Library/Caches/electron-builder
rm -rf ~/Library/Caches/node-gyp
rm -rf ~/Library/Caches/Yarn
rm -rf ~/Library/Caches/pnpm
rm -rf ~/Library/Caches/Homebrew
rm -rf ~/.npm/_cacache
rm -rf ~/.expo
rm -rf ~/.gradle/caches
rm -rf ~/.cocoapods/repos/trunk
brew cleanup --prune=all -s
df -h /System/Volumes/Data | tail -1
```

**Ganho típico:** 3–8 GB

---

## 4 · Claude, Chrome e Cursor — *risco zero*

Fecha os apps primeiro, senão o cache se regenera na hora. Login e abas do Chrome ficam intactos — só o cache vai embora.

```bash
osascript -e 'quit app "Google Chrome"'
osascript -e 'quit app "Cursor"'
sleep 3
rm -rf ~/Library/Application\ Support/Claude/Cache
rm -rf ~/Library/Application\ Support/Claude/Code\ Cache
rm -rf ~/Library/Application\ Support/Claude/GPUCache
rm -rf ~/Library/Application\ Support/Claude/DawnGraphiteCache
rm -rf ~/Library/Application\ Support/Claude/DawnWebGPUCache
rm -rf ~/Library/Application\ Support/Claude/logs
rm -rf ~/Library/Application\ Support/Claude/Service\ Worker/CacheStorage
rm -rf ~/Library/Application\ Support/Google/Chrome/*/Cache
rm -rf ~/Library/Application\ Support/Google/Chrome/*/Code\ Cache
rm -rf ~/Library/Application\ Support/Google/Chrome/*/GPUCache
rm -rf ~/Library/Application\ Support/Google/Chrome/*/Service\ Worker/CacheStorage
rm -rf ~/Library/Caches/Google
rm -rf ~/Library/Application\ Support/Cursor/Cache
rm -rf ~/Library/Application\ Support/Cursor/CachedData
rm -rf ~/Library/Application\ Support/Cursor/Code\ Cache
rm -rf ~/Library/Application\ Support/Cursor/GPUCache
df -h /System/Volumes/Data | tail -1
```

**Ganho típico:** 5–15 GB

---

## 5 · Sistema, logs e Lixeira — *risco zero*

Pede senha. Reinicie o Mac em algum momento depois — parte do que é limpo aqui é temporário de app em execução.

```bash
sudo rm -rf /Library/Caches/*
sudo rm -rf /private/var/log/*.gz
sudo rm -rf /private/var/log/asl/*.asl
sudo rm -rf /private/var/folders/*/*/C/*
sudo rm -rf /private/var/db/diagnostics/*
rm -rf ~/Library/Logs/*
rm -rf ~/.Trash/*
sudo periodic daily weekly monthly
df -h /System/Volumes/Data | tail -1
```

**Ganho típico:** 1–4 GB

---

## 6 · node_modules, Pods e builds — *risco médio*

Reinstala com `npm install` ou `pod install` quando for mexer em cada projeto. Não toca no código, só nas dependências baixadas.

```bash
rm -rf ~/projetos/*/node_modules
rm -rf ~/projetos/*/*/node_modules
rm -rf ~/projetos/*/ios/Pods
rm -rf ~/projetos/*/*/ios/Pods
rm -rf ~/projetos/*/.next
rm -rf ~/projetos/*/*/.next
rm -rf ~/projetos/*/.expo
rm -rf ~/projetos/*/*/.expo
df -h /System/Volumes/Data | tail -1
```

**Ganho típico:** 3–10 GB

---

## 7 · Archives do Xcode — *risco alto*

São os dSYMs dos builds que você publicou. Sem eles, um crash report de versão já na loja não é mais simbolizado. Só rode se topar perder isso.

```bash
rm -rf ~/Library/Developer/Xcode/Archives/*
df -h /System/Volumes/Data | tail -1
```

**Ganho típico:** 1–3 GB

---

## 8 · Instaladores e jogos — *risco alto*

Os `.dmg` já foram usados pra instalar. Modrinth e Minecraft só saem se você não joga mais — os mundos salvos vão junto.

```bash
rm -rf ~/Downloads/*.dmg
rm -rf ~/Library/Application\ Support/ModrinthApp
rm -rf ~/Library/Application\ Support/minecraft
rm -rf ~/Library/Caches/ModrinthApp
rm -rf ~/Library/Application\ Support/Any\ Video\ Converter
df -h /System/Volumes/Data | tail -1
```

**Ganho típico:** 3–8 GB

---

## 9 · Caçar o que sobrou — *não apaga nada*

O comando pra rodar quando o disco encher e você não souber por quê. Ele mostra o peso **fora da sua pasta pessoal**, que nenhuma limpeza de usuário alcança.

```bash
echo "=== FORA DA HOME ==="
sudo du -sh -x /Applications /Library /usr/local /opt /private/var 2>/dev/null | sort -rh
echo
echo "=== APPS MAIORES ==="
du -sh /Applications/* 2>/dev/null | sort -rh | head -20
echo
echo "=== ESPACO PURGAVEL ==="
diskutil info /System/Volumes/Data | grep -i -E "free|purge|used"
```

Foi esse comando que achou o **`Install macOS Tahoe.app` de 17 GB** encostado em `/Applications` depois de já instalado — o que ninguém procura porque não parece lixo.

---

## Depois de tudo

Reinicie o Mac. O Finder e os Ajustes continuam mostrando o número antigo de "Dados do sistema" até o `ApplicationsStorageExtension` recalcular, e ele só faz isso direito depois de um restart.

Pra não precisar repetir isso todo mês, o Espaço agenda um relatório diário que limpa sozinho o que é risco zero. Veja o [README](README.md).

---

## Erros que você pode encontrar

| Erro | O que é |
|---|---|
| `zsh: sure you want to delete all N files` | Faltou o bloco 0 |
| `Operation not permitted` | Falta Acesso Total ao Disco pro Terminal, em Ajustes → Privacidade e Segurança |
| `no matches found: .../*` | A pasta já está vazia. Não é problema |
| Espaço não muda depois de apagar | Foi pra Lixeira, ou o volume ainda não recalculou. Esvazie a Lixeira e rode o `df` de novo |
