https://neerc.ifmo.ru/wiki/index.php?title=CAP_%D1%82%D0%B5%D0%BE%D1%80%D0%B5%D0%BC%D0%B0

**CAP-теорема** — утверждение о том, что в распределённых системах нельзя одновременно добиться трёх свойств:

- **C**onsistency — на всех _не отказавших_ узлах одинаковые (с точки зрения пользователя) данные
- **A**vailability — запросы ко всем _не отказавшим_ узлам возвращают ответ
- **P**artition tolerance — даже если связь в системе стала нестабильной (вплоть до разделения системы на куски), но узлы работают, то система в целом продолжает работать

Формально мы это не формулировали и не доказывали. Оригинальная формулировка — Brewer's Conjecture (2000), а формализовано в работе Gilbert & Lynch (2004). Там есть много тонкостей с тем, что такое "не отказавший узел", "одинаковые данные", "разрыв связи", "система продолжает работать", "когда система всё-таки окончательно ломается и считается недоступной" и тому подобное.

Для общей эрудиции выглядят неплохо статьи[[1]](https://neerc.ifmo.ru/wiki/index.php?title=CAP_%D1%82%D0%B5%D0%BE%D1%80%D0%B5%D0%BC%D0%B0#cite_note-1)[[2]](https://neerc.ifmo.ru/wiki/index.php?title=CAP_%D1%82%D0%B5%D0%BE%D1%80%D0%B5%D0%BC%D0%B0#cite_note-2)

## Классификация алгоритмов

А вот алгоритмы, которые удовлетворяют хотя бы двум свойствам, можно нарисовать на диаграмма Венна:

[![Distributed-cap.png](https://neerc.ifmo.ru/wiki/images/thumb/9/9e/Distributed-cap.png/400px-Distributed-cap.png)](https://neerc.ifmo.ru/wiki/index.php?title=%D0%A4%D0%B0%D0%B9%D0%BB:Distributed-cap.png)

Дальше идут объяснения с лекции, они отличаются в разных источниках (например, где-то 2PC идёт вместе с Paxos в CP, а не в CA[[3]](https://neerc.ifmo.ru/wiki/index.php?title=CAP_%D1%82%D0%B5%D0%BE%D1%80%D0%B5%D0%BC%D0%B0#cite_note-3)), а где-то совпадают с лекцией[[4]](https://neerc.ifmo.ru/wiki/index.php?title=CAP_%D1%82%D0%B5%D0%BE%D1%80%D0%B5%D0%BC%D0%B0#cite_note-4), а где-то считают, что не-partition tolerance не нужно[[5]](https://neerc.ifmo.ru/wiki/index.php?title=CAP_%D1%82%D0%B5%D0%BE%D1%80%D0%B5%D0%BC%D0%B0#cite_note-5).

- Если мы хотим согласованность и доступность, то используем [протокол двухфазного коммита](https://neerc.ifmo.ru/wiki/index.php?title=2_Phase_Commit "2 Phase Commit"): он гарантирует нам согласованное состояние глобально во всей системе и мы всегда можем обслуживать запросы (если только не упал узел с соответствующими данными). Но если потерялась связь (а узлы не упали), то какие-то запросы нельзя обработать, потому что часть данных может быть в одной половине, а часть в другой. Каждый кусок всё ещё будет работать по отдельности, но глобальные транзакции выполнять мы не сможем.
- Иногда нам не так важна согласованность и мы согласны на простую eventual consistency — это когда информация может быть доступна не сразу везде, а только через какое-то время, если система здорова. Сюда идут [gossip-протоколы](https://neerc.ifmo.ru/wiki/index.php?title=Gossip-%D0%BF%D1%80%D0%BE%D1%82%D0%BE%D0%BA%D0%BE%D0%BB%D1%8B "Gossip-протоколы").
- Если мы хотим согласованность и толерантность к разделению, то надо жертвовать доступностью. Например, при помощи Paxos мы можем хранить все данные сразу на всех узлах, но тогда узлы, оказавшиеся в меньшинстве, ничего сделать не могут.
    

-  [http://blog.thislongrun.com/2015/04/the-unclear-cp-vs-ca-case-in-cap.html](http://blog.thislongrun.com/2015/04/the-unclear-cp-vs-ca-case-in-cap.html)
-  [https://habr.com/ru/post/322276/](https://habr.com/ru/post/322276/)
-  [http://www.cedanet.com.au/ceda/fallacies/cap-theorem.php](http://www.cedanet.com.au/ceda/fallacies/cap-theorem.php)
-  [http://book.mixu.net/distsys/single-page.html#the-cap-theorem](http://book.mixu.net/distsys/single-page.html#the-cap-theorem)
-  [https://codahale.com/you-cant-sacrifice-partition-tolerance/](https://codahale.com/you-cant-sacrifice-partition-tolerance/)