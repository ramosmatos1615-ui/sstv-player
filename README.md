# UnifiedTV – Player IPTV unificado (Android)

Carrega várias listas IPTV **ao mesmo tempo** (M3U por URL, M3U de arquivo local e Xtream Codes),
mescla tudo em **categorias unificadas** de Ao vivo, Filmes e Séries, com busca global,
cache local e player ExoPlayer (Media3) com troca automática de fonte.

## Instalação
1. Copie `UnifiedTV-1.2.0-AndroidTV.apk` para o celular, TV Box ou Android TV.
2. Permita "Instalar apps de fontes desconhecidas" e abra o APK.
3. Android 6.0+ (API 23). Funciona com toque e controle remoto (D-pad).

## Como funciona
| Recurso | Implementação |
|---|---|
| Múltiplas listas em paralelo | `SyncManager` – até 3 downloads simultâneos (coroutines + Semaphore) |
| M3U grande | `M3uParser` em streaming (Sequence linha a linha), gravação em lotes de 2000 |
| Xtream Codes | `XtreamClient` – `player_api.php`, JSON lido em streaming (`JsonReader`) |
| Mesclagem | `TextNormalizer.contentKey` remove prefixos de país, tags e qualidade: `BR \| ESPN HD`, `[BR] ESPN FHD` → `L:espn` |
| Categorias unificadas | `CategoryNormalizer` – regras por palavra-chave (Esportes, Notícias, Ação, Netflix…) |
| Detecção de tipo | `ContentClassifier` – pontuação por URL, extensão, duração do #EXTINF, padrões de episódio, ano, grupo, canais 24h e perfil estatístico do grupo (2 passadas) |
| Títulos | `MediaTitleParser` – S01E02, 1x02, T01 EP02, Temporada/Episódio, 1ª Temporada, Capítulo; ano de filmes; DUB/LEG/DUAL + 4K/FHD |
| União entre listas | "Matrix" e "Matrix (1999)" viram o mesmo filme; dublado e legendado viram fontes do mesmo título |
| Cache local | Room (SQLite WAL) + TTL de 12h + WorkManager a cada 12h; trocas atômicas por "geração" |
| Busca global | Tabela FTS4 (unicode61, sem acentos) com busca por prefixo em todas as listas |
| Player | Media3 ExoPlayer (HLS, DASH, TS, MP4/MKV), User-Agent/Referer por stream |
| Sem troca manual de lista | Cada item guarda todas as fontes; se uma falha, o player passa para a próxima |
| Prioridade | A ordem das listas define qual fonte é tentada primeiro |
| Favoritos | Toque longo em canal/filme; coração na tela da série |

No player: CH+/CH- (ou tecla N) alterna manualmente entre fontes do mesmo conteúdo.

## Compilar
```
./gradlew assembleRelease     # APK em app/build/outputs/apk/release/
./gradlew testReleaseUnitTest # testes de parser/normalização
```
Requer JDK 17 e Android SDK 35. A chave `app/unifiedtv.jks` (senha `unifiedtv`) é só para testes –
gere sua própria antes de distribuir.

## Android TV
- Aparece na tela inicial da TV (banner próprio) e funciona 100% com o controle remoto.
- Cards destacados e ampliados ao receber foco; OK abre, segurar OK favorita.
- No player: OK mostra os controles, CH+/CH- troca a fonte, Voltar sai.

## Agenda esportiva na Home (1.2.0)
- Agenda do dia integrada à Home, com jogos de futebol, NBA, NFL, MLB e NHL.
- Escudos, nomes dos times, placar, estado da partida e relógio/período ao vivo.
- Atualização automática dos placares a cada 60 segundos.
- Emissoras divulgadas pelo feed de placares aparecem como atalhos; ao selecionar, o app procura o canal correspondente nas listas IPTV ativas e abre o player existente.
- Os dados de jogos e escudos vêm do feed público ESPN. A disponibilidade de emissoras depende do que o feed informa e de existir um canal correspondente na lista IPTV do usuário.
