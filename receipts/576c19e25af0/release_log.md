# Release log: agentic-ai v1

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T14:53:18Z | 064760118ea2ee874c8e02aea5cc926b95000b97 | `{"candidate_tree": "sha256:a65af897dd8bbff9c12313dc98d197a9270870dbab46c61acda8946a75645e12", "gate_report": "82ea434051eb02c2c11d6aa82e4dc91c4a2aad183c623e0e08b9738064351e4c"}` |
| deposit-plan | 2026-11-09T14:53:19Z | 064760118ea2ee874c8e02aea5cc926b95000b97 | `{"archive_sha256": "afc95a555878bc922170875d8c28a8fec505cc1683e5dd0b8ecf0413976e971a"}` |
| deposit-draft | 2026-11-09T21:06:05Z | 064760118ea2ee874c8e02aea5cc926b95000b97 | `{"archive_sha256": "afc95a555878bc922170875d8c28a8fec505cc1683e5dd0b8ecf0413976e971a"}` |
| record-doi | 2026-11-10T10:50:14Z | 064760118ea2ee874c8e02aea5cc926b95000b97 | `{"CITATION.cff": "b17b002b9a9986620de66cc677d1f66abea58da3dd03a007a054ce7b6819da56", "README.md": "154598a6a61496a6906ffe18daf0a36a695307a12f6eddaefcc82c10ffcd4a1a", "RELEASE.yaml": "fad870a3e95caef1626c702bfb91ea3755ae42b5ecb745899ee347f4235ce0c2", "marketplace.json": "9b0cce4d92c7b9c2a49a7e1c8ce8f7d7b2fa3955f9eba52272807af184216ac7", "plugin.json": "9342fca9033faa1288247b00581eb72575c0e7a0b7c4265bcf14fb3f21d73137"}` |
| receipt | 2026-11-10T10:50:38Z | 064760118ea2ee874c8e02aea5cc926b95000b97 | `{"receipt": "621daf661e6324018ec5730ae65e08141f1519e59f56c0bcbcc7fa0487f6a97d"}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093110; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
