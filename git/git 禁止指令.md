## WARNING

```
git push -f   强制push，会使远程分支上的commit强制与当前分支一致
```

禁止：不要rebase集成分支，不要在本地rebase远程的commit，并且还push -f 到远程。这会使得历史消失。要明白，**历史是无法变更，除非你有修改历史的权力。**