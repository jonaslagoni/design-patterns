# design-patterns

## Roadmap

- [X] Add ECST pattern
- [X] Add CQRS pattern
- [X] Basic doc rendering
- [X] Github pages
- [ ] Event Sourcing
- [ ] Change Data Capture (CDC)

Communication Patterns
- [ ] Request-Reply
- [ ] point-to-point
- [ ] event streaming
- [ ] Publish-Subscribe pattern

Consumer Scalability / Patterns
- [ ] consumer groups
- [ ] partitioning
- [ ] Exclusive Consumer (HA)

Error Handling Patterns
- [ ] dead letter queue
- [ ] discard
- [ ] pause and retry
- [ ] saga


## Preview the website locally

```
python3 -m venv pyenv
source pyenv/bin/activate
pip install mkdocs-material
mkdocs serve
```

Windows
```
py -m venv pyenv
source pyenv/Scripts/activate
pip install mkdocs-material
mkdocs serve
```

Open http://127.0.0.1:8000

> Doc rendering is using [mkdocs material](https://squidfunk.github.io/mkdocs-material/).

