https://stackoverflow.com/questions/44204973/difference-between-exactly-once-and-at-least-once-guarantees

Below definitions are quoted from [Akka](http://doc.akka.io/docs/akka/current/scala/general/message-delivery-reliability.html) Documentation

**at-most-once** **delivery**

> means that for each message handed to the mechanism, that message is delivered zero or one times; in more casual terms it means that messages may be lost.

**at-least-once** **delivery**

> means that for each message handed to the mechanism potentially multiple attempts are made at delivering it, such that at least one succeeds; again, in more casual terms this means that messages may be duplicated but not lost.

**exactly-once** **delivery**

> means that for each message handed to the mechanism exactly one delivery is made to the recipient; the message can neither be lost nor duplicated.

The first one is the cheapest—highest performance, least implementation overhead—because it can be done in a fire-and-forget fashion without keeping state at the sending end or in the transport mechanism. The second one requires retries to counter transport losses, which means keeping state at the sending end and having an acknowledgement mechanism at the receiving end. The third is most expensive—and has consequently worst performance—because in addition to the second it requires state to be kept at the receiving end in order to filter out duplicate deliveries