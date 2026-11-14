# finviral

[![finviral â€” Radar Financeiro](guia/assets/banner.jpg)](https://inematds.github.io/finviral/guia/)

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

## O que Ã©

O finviral Ã© um sistema que faz, sozinho, vÃ­deos curtos (9:16) sobre o que os maiores influenciadores de finanÃ§as do Brasil disseram nos vÃ­deos deles. Ele assiste aos vÃ­deos do YouTube por transcriÃ§Ã£o, separa os nÃºmeros, os fatos e as polÃªmicas, e monta um vÃ­deo com nÃºmeros grandes animados, a fonte de cada dado na tela e o Nei comentando (avatar). Serve para quem quer acompanhar o mercado em um minuto, sem recomendaÃ§Ã£o de investimento. Roda num computador com GPU, com a assinatura do Claude Code e do HeyGen.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/finviral/guia/**

## Como funciona

1. **Coleta:** lista os vÃ­deos dos canais de `canais.json` (sÃ³ metadados, yt-dlp), baixa o Ã¡udio e transcreve localmente (inemavox, Whisper large-v3).
2. **Roteiro:** o `claude -p` (assinatura, sem API) lÃª as transcriÃ§Ãµes e grava trÃªs arquivos:
   - `base.md`: cada nÃºmero, quem disse, o vÃ­deo e o trecho literal;
   - `roteiro.json`: falas e visuais;
   - `legenda.txt`: o texto para aprovaÃ§Ã£o.
3. **ConferÃªncia (`auto/checar.py`):** confere se cada trecho do `base.md` estÃ¡ de fato na transcriÃ§Ã£o. TambÃ©m confere se os nÃºmeros da fala estÃ£o por extenso, se as imagens existem, se o CTA estÃ¡ presente e se aparece "nÃ£o Ã© recomendaÃ§Ã£o". Se reprovar, o roteiro volta uma vez para correÃ§Ã£o. Se reprovar de novo, o vÃ­deo nÃ£o Ã© feito.
4. **Avatar:** o HeyGen gera o avatar do Nei pelo estÃºdio (assinatura, sem API de geraÃ§Ã£o).
   - O `motor/retempo.py` alinha a arte Ã  fala do avatar.
   - Se o HeyGen falhar, sai a versÃ£o com a foto e a voz do Nei (Chatterbox).
5. **VÃ­deo:**
   - O `motor/palco.html` Ã© renderizado quadro a quadro pelo Playwright, com contadores, grÃ¡fico, carimbo de polÃªmica, legenda palavra a palavra e fonte em toda cena.
   - O avatar fica no canto, com trilha.
6. **PublicaÃ§Ã£o:** o vÃ­deo Ã© publicado no YouTube Radar Viral (lives24) pelo `yt-pubx` (`auto/publicar.sh`) e vai sempre ao bot v3 no Telegram, com o link. Se o avatar falhar, a versÃ£o com foto sÃ³ vai ao bot.
7. **SÃ¡bado e domingo â€” o longo 16:9:** depois do curto, sai "A semana explicada" (5 a 7 min, um capÃ­tulo por influenciador), com as cenas animadas do explicavideo v2 e peÃ§as de finanÃ§as (`motor/longo.html`, `auto/roteiro-longo.md`, `auto/checar.py --longo`, `auto/longo.sh`). O curto avisa no fim que a explicaÃ§Ã£o completa estÃ¡ no INEMA TIA.

## Agenda (systemd, usuÃ¡rio)

| Quando | Rodada | O que sai |
|---|---|---|
| Segunda a sexta, 02:00 | `diario` | 40â€“60 s com o que eles publicaram ontem e anteontem |
| SÃ¡bado, 02:00 | `semanal` | 90 s a 2 min: balanÃ§o da semana + longo 16:9 "A semana explicada" (5â€“7 min) |
| Domingo, 02:00 | `domingo` | ~90 s: o que eles esperam para a semana que vem + longo 16:9 explicado |

O bot v3 sempre recebe uma de trÃªs coisas: o vÃ­deo (com o link do lives24), "sem pauta" (dia fraco nÃ£o vira vÃ­deo) ou "falhou em X".

```bash
cp systemd/finviral-* ~/.config/systemd/user/ && systemctl --user daemon-reload
systemctl --user enable --now finviral-diario.timer finviral-semanal.timer finviral-domingo.timer
systemctl --user list-timers 'finviral-*'
```

## Avulso (fora da agenda)

```bash
auto/rodar.sh diario 2026-10-09          # rodada de um dia especÃ­fico
SEM_AVATAR=1 SEM_BOT=1 auto/rodar.sh semanal   # teste: sem HeyGen e sem enviar
echo "2026-10-17 diario" >> auto/pular.txt     # pular uma rodada
```

Para uma ediÃ§Ã£o feita Ã  mÃ£o (como `edicoes/2026-10-09-semana`):

```bash
E=edicoes/<data>-<slug>
python3 motor/narrar.py $E 1.12 && python3 motor/gerar_cena.py $E && motor/montar.sh $E $E/video.mp4   # voz do Nei + foto
motor/avatar_heygen.sh $E FINVIRAL-<titulo> && python3 motor/gerar_cena.py $E --avatar && motor/compor_avatar.sh $E $E/video.mp4   # avatar
```

## Influenciadores

A lista fica em `canais.json`, com `"ativo": false` para pausar. Hoje Ã© o Top 10 do ranking Anbima FInfluence (10Âª ediÃ§Ã£o): Bruno Perini, Economista Sincero, Ricardo Amorim, O Primo Rico, Hermann Greb, CafÃ© com Ferri, Pablo Spyer, Primo Pobre, Nathalia Arcuri (Me Poupe!) e Tiago Reis.

Se aparecer em destaque outro nome consolidado e confiÃ¡vel, o sistema sugere a inclusÃ£o na mensagem do bot. Nome emergente nunca entra, e a inclusÃ£o sÃ³ acontece com o ok do Nei.

## Regras de cada vÃ­deo

- Quadro 0 cheio: data dd/mm/aaaa e o nÃºmero mais forte.
- SÃ³ entra nÃºmero que estÃ¡ na transcriÃ§Ã£o, com a fonte (canal, data, tÃ­tulo) na tela. NÃ£o Ã© recomendaÃ§Ã£o de investimento.
- EleiÃ§Ã£o e polÃ­tica: o vÃ­deo relata o que cada um disse, sempre atribuÃ­do, sem tomar lado.
- Fala no tom do Nei, tratando por "vocÃª", com nÃºmeros e anos por extenso.
- Fim: gancho de comentÃ¡rio e CTA de gestÃ£o de agentes de IA no inema.club.
- HeyGen sÃ³ pelo estÃºdio da assinatura. A oferta de troca de plano Ã© sempre recusada.

## Requisitos

- yt-dlp
- inemavox (`~/projetos/inemavox`)
- cardshorts (voz Chatterbox `nei`)
- Node com Playwright (`motor/node_modules` â†’ link para o do eventosextremospub)
- ffmpeg
- `claude` (Claude Code logado na assinatura)
- Xvfb `:99` e o perfil do HeyGen do explicavideos
- bot v3 (`refazvideo/ferramentas/enviar_bot_v3.mjs`)

LicenÃ§a MIT. O conteÃºdo citado pertence aos canais de origem, sempre creditados na tela.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe