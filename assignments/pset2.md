# Problem Set 2: Composing Concepts
## Concept Questions
1. Contexts in the *NonceGeneration* concept ensure that uniqueness is enforced only within a specific subset of generated strings, rather than globally. This avoids unnecessary conflicts between independent groups of nonces. In the case of URL shortening, the context would be the shortURLBase (the domain or base URL). This way, two short URLs can reuse the same suffix as long as they belong to different bases/domains.

2. In the specification, the set of used strings expresses the principle that each generated string must be unique within a context. By recording all previously generated strings, the spec makes it explicit that generate cannot return one already in the set. This is the simplest way to formalize the uniqueness requirement at the abstract level. For the counter-based implementation, this set of used strings corresponds all strings that correspond to numbers less than or equal to the current counter value. The abstraction function maps the counter n to the set { "0", "1", …, str(n)}.

3. The main advantage is that short URLs would be easier to remember, pronounce, and type, making them more user-friendly and trustworthy (people are more likely to trust a link made from a real word than from a random string of characters). A disadvantage is that the chosen words may have no connection to the actual target URL, which can feel confusing or misleading. To support this scheme in the concept, the NonceGeneration state would need to include a dictionary (a set of available words) in addition to the set of already used words for each context. Also, the generate action would select a word from this dictionary rather than from the space of arbitrary string nonces, while still ensuring uniqueness within each context.

## Synchronization Questions
1. In the first sync, only shortUrlBase is included because generating a nonce requires only the context, not the target URL. In the second sync, both targetUrl and shortUrlBase are needed, since the register action needs both the base and the long URL that the new short URL should map to.

2. The omission convention is only safe when the argument or result name is identical to the bound variable name. When they differ, or when it may be unclear, both must be written explicitly to show how values are passed and bound, especially across different concepts. For example, in the register sync, the result of NonceGeneration.generate is bound to the variable nonce but passed as the argument shortUrlSuffix to UrlShortening.register. Since the names differ, both should be written to make this connection clear.

3. The request action appears in the first two syncs because those behaviors are initiated by the user’s call/requests to Request.shortenUrl. The third sync has no request action since it represents an automatic system action. Once a short URL is registered, the system itself sets the expiry without user involvement.

4. If the application always used a fixed domain name, we would remove shortUrlBase from the syncs and instead hardcode the constant domain (ex. "bit.ly"). For example, for register, the when clause would have Request.shortenURL(targetURL) instead and the then clause would have UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: "bit.ly", targetUrl) or whichever specific base you are using.

5.   **sync**  expireLink<br>
     **when**  ExpiringResource.expireResource(): (resource: shortUrl) <br>
     **then**  URLShortening.delete(shortURL)

## Extending the design
1. **concept** analyticsTracking [Item] <br>
     **purpose** record how many times an item is accessed <br>
     **principle** after creating a record for an item, every access of it increments its count <br>
     **state** <br>
     &nbsp;&nbsp;&nbsp;
     a set of Records with <br>
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     an Item <br>
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     a counter Number <br>
     **actions** <br>
     &nbsp;&nbsp;&nbsp;
    addRecord(item: Item): (record: Record)<br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **requires** there is no existing Record for this item <br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **effects** creates new Record for this item with counter set at 0<br>
    &nbsp;&nbsp;&nbsp;
    recordAccess(item: Item):<br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **requires** there is an existing Record for this item <br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **effects** increments the counter for this item's record<br>
    &nbsp;&nbsp;&nbsp;
    deleteRecord(item: Item):<br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **requires** there is an existing Record for this item <br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **effects** deletes the Record for this item<br>

    **concept** ItemOwnership [User, Item] <br>
     **purpose** restrict access of an item to specific users <br>
     **principle** after assigning a user to an item, only the assigned users may be authorized for that item<br>
     **state** <br>
     &nbsp;&nbsp;&nbsp;
     a set of Items with <br>
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
     a set of Users <br>
     **actions** <br>
     &nbsp;&nbsp;&nbsp;
    assign(user: User, item: Item): <br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **effects** add user to item's set of users<br>
    &nbsp;&nbsp;&nbsp;
    authorize(user:User, item: Item): <br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **requires** user is in item's set of users <br>
    &nbsp;&nbsp;&nbsp;
    unassign(user:User, item: Item): <br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **requires** user is in item's set of users <br>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    **effects** user is removed from item's set of users <br>
2.   **sync**  registerAnalytics<br>
     **when**
     UrlShortening.register(): (shortUrl) <br>
     **where** user is the user creating the shortened URL<br>
     **then** <br>
     &nbsp;&nbsp;&nbsp;
     analyticsTracking.addRecord(item: shortUrl): (record)<br>
     &nbsp;&nbsp;&nbsp; ItemOwnership.assign(user, item: shortUrl)<br>

     **sync**  recordLookup<br>
     **when** UrlShortening.lookup (shortUrl): (targetUrl) <br>
     **then** analyticsTracking.recordAccess(item: shortUrl)

      **sync**  examineAnalytics<br>
     **when**<br>
     &nbsp;&nbsp;&nbsp;
     Request.getAnalytics (user, shortUrl)<br>
     **where** user is requesting the analytics and count is the counter for the shortUrl in analyticsTracking <br>
     **then**<br>
     &nbsp;&nbsp;&nbsp;
     ItemOwnership.authorize (user, item: shortUrl) <br>
     &nbsp;&nbsp;&nbsp;
     return count to user
3. **Feature Requests**
    - Allowing users to choose their own short URLs
        - How: add an additional sync for register so that when Requests.shortenURL has a shortURLSuffix parameter, URLShortening.register is called with that suffix directly instead of calling NonceGeneration in between.
    - Using the “word as nonce” strategy to generate more memorable short URLs
        - How: Add a dictionary of words for the NonceGeneration concept. Add an action that generates unused strings from this dictionary, alongside the existing action for arbitrary strings. Users (or the system) could then select which action to invoke.
    - Including the target URL in analytics
        - How: In the registerAnalytics sync, do anylyticsTracking.addRecord(item: targetUrl) instead. Also, in the recordLookup sync, do analyticsTracking.recordAccess(item:targetUrl) instead of shortUrl, so counts are aggreagated for any access to targetUrl from multiple short URLs. Would list targetUrl as one of the arguments for Urlshortening.register. For examineAnalytics you would return the count for the targetUrl in analyticsTracking (could find using lookup action).
    - Generate short URLs that are not easily guessed
        - How: modify the NonceGeneration concept so that there is a secureGenerate action that may have a stricter postcondition relating to how to make a harder-to-guess nonce.
    - Supporting reporting of analytics to creators of short URLs who have not registered as user
        - Undesirable: This feature should not be included. Analytics may expose sensitive information about traffic volume or demographics, which should only be visible to authorized users. Providing analytics to unregistered creators undermines privacy and security and weakens the ownership model.
