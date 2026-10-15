# Hyperframes

Open-source video rendering framework: write HTML, render video.

## Skills

This repo ships AI agent skills via [vercel-labs/skills](https://github.com/vercel-labs/skills). Install them before writing compositions — they encode framework-specific patterns that generic docs don't cover. **Default to the core set** — the `/hyperframes` router installs each creation workflow on demand; install everything only when the user explicitly asks for the full set.

```bash
npx hyperframes skills update           # default: installs/refreshes the core set — workflows install on demand
npx hyperframes skills                  # all 21 published skills at once
npx skills add heygen-com/hyperframes   # interactive picker (terminal only; repo-internal skills are excluded by default)
```

**Creation workflows** route through one entry skill — read `/hyperframes` first: it orients you to the whole surface, confirms the brief up front (the intent layer), and maps "make me a…" intent — usually a video, but also a navigable deck (`/slideshow`) or a composition port (`/remotion-to-hyperframes`) — to a concrete workflow. Consult it before invoking a specific workflow:

- `/product-launch-video` — any **website** URL (or a pre-written script / text brief in no-capture mode) → a product launch / promo video, or a site tour / showcase featuring the site's own captured screens; up to ~3 min (sweet spot ~30-90s).
- `/faceless-explainer` — arbitrary text, **no URL and no website capture** → faceless explainer, up to ~3 min (sweet spot ~30-90s); every visual is LLM-invented (typography / abstract graphics / diagram / data-viz).
- `/embedded-captions` — an existing talking-head video (MP4) → the same footage with captions / subtitles added (verbatim rail + embedded climax, or pure-cinematic embed); the footage itself is untouched (no NLE-style editing).
- `/talking-head-recut` — an existing talking-head / interview / podcast video (MP4) → the same footage packaged with designed **graphic overlays** (kinetic titles, lower-thirds, data callouts, pull-quotes, side panels, pip) synced to the transcript; the clip plays unchanged underneath, footage untouched. For plain captions/subtitles → `/embedded-captions`.
- `/pr-to-video` — a GitHub PR (URL / `owner/repo#N` / "this PR") → code-change explainer, up to ~3 min (changelog / feature reveal / fix / refactor). A PR link, not a product website.
- `/motion-graphics` — a short (typically under 10s) design-led **motion graphic**, motion-is-the-message, no narration: kinetic type, a stat / number count-up, a chart, a logo sting, a lower-third / overlay, or an animated tweet / headline / captured-page highlight; rendered to MP4 or a transparent overlay. Longer / narrated / custom → `/general-video`.
- `/music-to-video` — a **music track** (audio file, video to pull audio from, or one generated from a mood brief) → beat-synced video (lyric / slideshow / kinetic promo). Music drives pacing; user-supplied images / videos are cut onto the same beat grid.
- `/slideshow` — a **presentation / pitch deck / interactive deck** — discrete slides, fragment reveals, branching, hotspot navigation, presenter mode. Output is a navigable deck, not a rendered video.
- `/general-video` — fallback for any other video creation (title card, longer brand / sizzle reel, multi-scene montage, static loop, custom composition) and the home of **companion mode** — co-create with the full HyperFrames toolbox; the original hyperframes flow — design → plan → layout → build → validate, any length.

**Porting an existing composition?** `/remotion-to-hyperframes` translates a Remotion (React) video composition into HyperFrames HTML — a source migration, separate from the creation workflows above.

## Issue and PR triage

Read [TRIAGE.md](TRIAGE.md) before classifying issues, advertising contribution work, or triaging PRs. Check current source and overlapping PRs; keep difficulty, readiness, and ownership separate. Apply changes only within the task's authorization and verify the resulting GitHub state.

## Build & Test

```bash
bun install     # Install dependencies (NOT pnpm — do not create pnpm-lock.yaml)
bun run build   # Build all packages
bun run test    # Run all tests
```

Packages load each other through their `exports`. Under Node, core, parsers,
lint and studio-server resolve to `dist/`, and player always loads its built
bundle. Run `bun run build` once in a fresh worktree, before any package's
tests or typecheck. After editing a package that another package imports, run
that package's `build` again before the dependent package's tests. Core's
`src/generated/` is git-ignored build output: if you see
`Cannot find module '../generated/runtime-inline'`, run
`bun run build:hyperframes-runtime`.

One file: `bun run --cwd packages/<pkg> test <path>`. Each package's `test`
script picks its runner (vitest in most, `bun test` in aws-lambda and
gcp-cloud-run). Producer is the exception: a file that imports `bun:test` runs
with `bun test <file>`, any other with `bunx vitest run <file>`. Root `scripts/*.test.mjs` run through `bun run test:scripts`,
which is an explicit list in `package.json`. CI's test-reachability check fails
any test file that no runner reaches.

### Linting & Formatting

Uses **oxlint** and **oxfmt** (not eslint, not prettier, not biome).

```bash
bunx oxlint <files>        # Lint
bunx oxfmt <files>         # Format
bunx oxfmt --check <files> # Check formatting (CI / pre-commit)
```

Always lint and format changed files before committing. Lefthook pre-commit hooks enforce this automatically.

### Composition Validation

After creating or editing any `.html` composition:

```bash
npx hyperframes lint       # Static HTML structure check
npx hyperframes check      # Browser gate (headless Chrome — runtime errors, layout, motion, WCAG contrast)
```

Both must pass before previewing or considering work complete.

## Project Structure

```
packages/
  cli/                  → hyperframes CLI (create, preview, lint, render)
  core/                 → Types, parsers, generators, linter, runtime, frame adapters
  engine/               → Seekable page-to-video capture engine (Puppeteer + FFmpeg)
  player/               → Embeddable <hyperframes-player> web component
  producer/             → Full rendering pipeline (capture + encode + audio mix)
  shader-transitions/   → WebGL shader transitions for compositions
  studio/               → Browser-based composition editor UI (read packages/studio/AGENTS.md first)
registry/
  blocks/               → Installable sub-composition scenes (50+)
  components/           → Installable effects and snippets
  examples/             → Starter project templates
docs/                   → Mintlify documentation site (hyperframes.heygen.com)
skills/                 → AI agent skill definitions
```

## Key Conventions

- **Package manager**: bun (not pnpm, not npm for workspace operations)
- **Commit format**: Conventional commits (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`)
- **TypeScript**: Avoid `any` and `as T` assertions. Prefer type guards and narrowing.
- **Compositions**: HTML files with `data-*` attributes. Clips need `class="clip"`. Register one paused GSAP root timeline per composition on `window.__timelines`. Scene timelines manually added to that root must not be paused, or they will not advance when the root is seeked.
- **Frame Adapters**: Animation runtimes plug in via the seek-by-frame adapter pattern. GSAP is the primary adapter.
- **Deterministic rendering**: No `Date.now()`, no unseeded `Math.random()`, no render-time network fetches.

## Documentation

- Docs: https://hyperframes.heygen.com/introduction
- Catalog (50+ blocks): https://hyperframes.heygen.com/catalog/blocks/data-chart

## Preferências permanentes — edição de vídeos para redes sociais

Este projeto é o ambiente de edição de vídeos do usuário, com foco em Instagram. As instruções originais do HyperFrames acima continuam válidas. Aplique as preferências abaixo aos próximos trabalhos; pedidos específicos do usuário prevalecem sobre os padrões.

### Atendimento e fluxo de edição

- Comunique-se em português. Use os vídeos enviados pelo usuário como material de origem e preserve os arquivos originais.
- Leia `skills/hyperframes/SKILL.md` como ponto de entrada e selecione as skills de acordo com a edição solicitada. Consulte as instruções locais antes de escrever composições.
- Para cortes, montagem e reorganização de trechos, use `general-video` e `hyperframes-core`. Para reenquadramentos e zooms animados, acrescente `hyperframes-keyframes`.
- Use `embedded-captions` para legendas sobre filmagem preservada; `talking-head-recut` para elementos gráficos sobre filmagem preservada; `hyperframes-audio` para mixagem; `media-use` para transcrição, tratamento e preparação de mídia; e `hyperframes-cli` para validação e exportação.
- Inspecione duração, resolução, orientação, taxa de quadros e áudio dos arquivos recebidos antes de editar. Não invente transcrições; confira nomes, números e sincronismo das legendas.
- Aproveite as instruções já fornecidas. Pergunte apenas pelo que faltar para definir a edição, como objetivo, duração ou trechos essenciais; resolva escolhas técnicas rotineiras sem confirmações repetidas.
- Preserve o sentido das falas e a continuidade. Escolha ritmo, transições, textos e efeitos conforme o conteúdo e o pedido; não aplique um estilo visual ou musical fixo a todos os trabalhos.

### Padrão de entrega para Instagram Reels

- Exporte em **MP4, preferencialmente 1080 × 1920 pixels, vertical 9:16**, salvo pedido diferente. Use vídeo H.264, pixel format `yuv420p`, áudio AAC quando houver áudio e `faststart` quando suportado.
- Use 30 fps como padrão; preserve ou ajuste a taxa conforme a fonte e a intenção do projeto. Não prometa aumentar o detalhe de fontes de baixa resolução por meio de upscale.
- Reenquadre sem distorcer a imagem, mantendo rostos, ações e informações essenciais visíveis. Quando o corte vertical prejudicar o conteúdo, use uma composição que preserve essas informações.
- Mantenha títulos, legendas e chamadas legíveis no celular e afastados das bordas e das áreas ocupadas pelos controles do Instagram. Verifique a posição no enquadramento final.
- Quando houver legendas, mantenha bom contraste, tamanho legível, quebras naturais e sincronismo com a fala. Quando houver música, preserve a inteligibilidade da voz e evite clipping.

### Organização, validação e continuidade

- Organize cada trabalho em `videos/<nome-do-projeto>/`, caminho já ignorado pelo Git, separando originais, composição, recursos e exportações. Não sobrescreva exportações anteriores sem necessidade; use nomes claros com versões.
- Registre o briefing e as decisões de edição no projeto, seguindo os arquivos previstos pela skill escolhida, para permitir retomadas e revisões.
- Mantenha trabalhos de vídeo separados do código do framework. Use o checkout existente; não crie worktrees sem pedido explícito.
- Neste ambiente de nuvem, quando disponível, ative as ferramentas com `source /workspace/.tools/hyperframes-env.sh`. Consulte as instruções salvas de inicialização se precisar reiniciar o Studio; processos não são persistentes entre máquinas.
- Execute `hyperframes lint` e `hyperframes check` no projeto após criar ou alterar composições HTML, conforme as instruções originais. Investigue falhas antes de considerar a composição pronta.
- Após exportar, confira com `ffprobe` o contêiner, codecs, resolução, duração, taxa de quadros e presença de áudio esperada. Verifique quadros representativos e o sincronismo do áudio e das legendas antes da entrega; relate qualquer verificação que não pôde executar.
- Entregue um link para o MP4 gerado, com uma descrição breve das alterações e das características do arquivo. Não declare sucesso antes de verificar o arquivo final.
- Não publique nas redes sociais nem envie arquivos a terceiros sem instrução explícita. Se uma etapa exigir credenciais ou serviço externo, verifique o que já está disponível e solicite apenas o requisito ausente, sem pedir segredos em chat.
