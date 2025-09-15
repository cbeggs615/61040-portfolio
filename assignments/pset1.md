# Problem Set 1: Reading and Writing Concepts

## Exercise 1: Reading a concept
### Question 1: Invariants
1. For each item in a registry, the sum of all purchase counts for that item must not exceed the requested count for that item.
2. Every purchase must be linked to an existing request in the registry.

The first invariant is more important because it represnts the core logic of the gift registry, which is preventing over-purchasing of requested items. If this invariant is broken, givers could overbuy certain items. The purchase action is the one most affected by this invariant. It preserves this invariant through its precondition "has a request for this item with at least count" and by decrementing the request count when a purchase is made. This ensures the remaining available count accurately reflects what can still be purchased.

### Question 2: Fixing an action
The removeItem action could potentially break this invariant. Assume a request for Item A is created with count 5 by the owner in an active registry. A giver purchases 3 of Item A. Now the request count for Item A is 2. Then, the owner of the registry removes the request of Item A. Now, there is no requested count for Item A. Thus, the sum of all purchase counts of Item A is 3 but the requested count is 0.

This problem could be fixed by adding that removeItem also requires that no purchases exist for that item in the registry.

### Question 3: Inferring Behavior
A registry can be opened and closed repeatedly according to the specs. A reason to allow this could be if an owner of a registry opened the registry once for close friends, then closed it to see what was bought and adjust remaining requests or add additional requests before reopening at a later date for more people.

### Question 4: Registry deletion
The lack of a delete registry action could matter in practice since users may want to delete no longer active registries for better management, to avoid confusion, and to have a clean workspace. Also, the accumulation of all this data without the ability to delete it could lead to performance issues and also privacy concerns.

### Question 5: Queries
1. The owner of the registry would see what items have been purchased, how many, and by who.
2. A giver would query to see what items are still needed and what quantity are still requested.

### Question 6: Hiding purchases
In order to allow certain registries to hide purchases, we could add a showPurchases flag to the Registry type. So each Registry would have a User, and active Flag, a showPurchases Flag, and a set of Requests. If this flag is not true, the owner of the registry wouldn't be able to see the Purchases associated with the Requests of this Registry.

### Question 7: Generic Types
Keeping User and Item as generic parameters allows this concept to be focused on only tracking requested and purchased quantities, so its better for separation of concerns. Item or User details could be specified by other concepts such as authentification or a product catalog concept. Also, it allows this concept to be used more generically for different Item systems, say for different companies or uses.

## Exercise 2: Extending a familiar concept
### Question 1
**state**<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a set of Users with...<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a username String <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a password String <br>
### Question 2
**actions**<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;register (username: String, password: String): (user: User)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** no User exists with that username <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** creates and returns a new User with username and password<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;authenticate (username: String, password: String): (user: User)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** a User with this username exists and its password matches the inputted password <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** returns that User

### Question 3
An essential variant of the state is that no two Users can have the same username. It is preserved since the only way to creater a User (the register action) requires that no User with that username already exists.

### Question 4
**state**<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a set of Users with...<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a username String <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a password String <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a confirmed Flag <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a secretToken String <br>
**actions**<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;register (username: String, password: String): (user: User, secretToken: String)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** no User exists with that username <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** creates and returns a new User with username and password with confirmed set to False and a new secretToken<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;authenticate (username: String, password: String): (user: User)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** a User with this username exists, its password matches the inputted password, this User is confirmed <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** returns that User<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;confirm (username: String, secretToken: String): (user: User)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** a User with this username exists and its secretToken matches the inputted token and this User's confirmed flag is False <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** set User's confirmed flag to true, returns that User

## Exercise 3: Comparing concepts
### The concept
**concept** PersonalAccessToken [User, Scope] <br>
**purpose** provide scoped, revocable authentication for access <br>
**principle** user creates multiple tokens with specific scopes and expiration dates;
each token can be used independently to authenticate certain operations
within its granted permissions;
tokens can be individually revoked without affecting other tokens or the user's password <br>
**state** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a set of Tokens with ... <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
an owner User <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a tokenString String <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a set of Scopes <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
an expiration Date<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a name String<br>
**actions** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
createToken (user: User, scopes: set of Scopes, expiration: Date, name: String): (token: Token) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** expiration is after current date<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** create a new token with given scopes, expiration, and name for user with a new generated tokenString <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
authenticate (username: String, tokenString: String): (user: User, scopes: set of Scopes) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** a token exists with this tokenString,
            and current date is before the token's expiration, username is not blank<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** return the token's owner and scopes<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
revokeToken (user: User, token: Token) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  **requires** user owns this token, token exists<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  **effects** remove the token from the set of Tokens

### Analysis
This concept is different from PasswordAuthentication since with tokens, you are not creating a new user or password. Rather, tokens are like specialized logins for different actions or permissions associated with your existing account.

This is what I'd add to the very top of the GitHub documentation to make this difference more clear.
>Personal access tokens are not passwords. A password grants full access to your account. A token is a limited credential. Each token can be tied to specific permissions and set to expire automatically. You can create multiple tokens for different tools or scripts, revoke them individually, and reduce the risk of exposing your main password.

## Exercise 4: Defining familiar concepts
### URL Shortener
**concept** URLShortener [User] <br>
**purpose** create short, memorable aliases for long URLs <br>
**principle** a user provides a long URL and optionally a custom suffix;
the system creates a mapping from a short URL (with either the custom suffix or an auto-generated one) to the original long URL;
when anyone accesses this short URL, they are redirected to the original long URL<br>
**state** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a set of Mappings with ... <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a shortSuffix String <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a longURL String <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
an owner User <br>
**actions** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
createCustomMapping (owner: User, longURL: String, customSuffix: String): (shortURL: String) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** customSuffix is not empty, no mapping exists with this customSuffix<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** create a new mapping with this customSuffix and longURL for owner;
             return the short URL formed by combining base domain with shortSuffix <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
createAutoMapping (owner: User, longURL: String): (shortURL: String)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** create a new mapping with an auto-generated unique shortSuffix and longURL for owner;
             return the short URL formed by combining base domain with generated suffix<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
resolve (shortSuffix: String): (longURL: String) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  **requires** a mapping exists with this shortSuffix<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  **effects** return the longURL from the matching mapping<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
deleteMapping (user: User, shortSuffix: String) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  **requires** a mapping exists with this shortSuffix and user matches corresponding owner<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  **effects** remove the corresponding mapping<br>

  >**Notes**: createCustomMapping handles user-specified suffixes with uniqueness validation, while createAutoMapping generates guaranteed-unique suffixes automatically. All mappings require a owner user, enabling tracking of user's active mappings and access control (only owners can delete their mappings). Resolution requires no authentication since short URLs are meant to be publicly accessible. Storing only suffixes rather than complete URLs keeps the concept independent of specific domains (tinyurl.com, bit.ly, etc.). Full short URLs are constructed only in return values. Custom suffixes prevent collisions through uniqueness requirements. Auto-generated suffixes achieve uniqueness through the generation process (implementation retries until unique). The concept focuses purely on suffix-to-URL mapping. It doesn't validate URL functionality or handle features like expiration policies, which vary widely between services and can be added as extensions.

### Conference Room Booking
**concept** RoomReservation [User, Room] <br>
**purpose** reserve conference rooms for specific time periods to prevent conflicts <br>
**principle** a user requests to book a room for a specific time period;
if the room is available during that entire period, a reservation is created;
no other user can book the same room for any overlapping time period;
the user can later cancel their reservation, making the room available again<br>
**state** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a set of Bookings with ... <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a Room <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a booker User <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a start Time <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
an end Time <br>

**actions** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
makeBooking (booker: User, room: Room, startTime: Time, endTime: Time): (booking: Booking) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** startTime is before endTime, startTime is in the future, no booking exists for this room that overlaps with [startTime, endTime)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** create a new booking for room for the booker with startTime and endTime <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
cancelBooking (user: User, booking: Booking) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  **requires** booking exists, user is the booker of this booking<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  **effects** remove the booking from the set of Bookings<br>
  >**Notes**: The concept uses half-open intervals [startTime, endTime) where startTime is inclusive and endTime is exclusive. This prevents edge case conflicts when one meeting ends exactly when another begins. The constraint "startTime is in the future" prevents booking rooms in the past, which would not make sense for a reservation system. Only the original booker can cancel their booking. In practice, administrators might also need cancellation rights. The concept doesn't enforce minimum or maximum booking durations, leaving such policy decisions to other logic or extensions. Both User and Room are generic parameters, allowing the concept to work with different user systems and room identification schemes. It is assumed that this concept is queryable so, for example,  users could see available rooms for certain times and which times are/are not reserved for a given room.

### Billable Hours Tracking
**concept** BillableHours [User, Project] <br>
**purpose** track time spent on client projects for billing and record keeping purposes <br>
**principle** an employee starts a work session by specifying a project and work description;
the system records the start time and keeps the session active;
when the employee ends the session, the system records the end time and calculates billable hours;
if a session is forgotten and left open, it can be manually closed with a specified end time<br>
**state** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a set of Sessions with ... <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
an employee User <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a project Project <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a start Time <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
an end Time <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a currentlyActive Flag <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
a description String <br>

**actions** <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
createSession (employee: User, project: Project, description: String): (session: Session)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** no active session exists for this employee <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** create a new currentlyActive session for employee for project, use current time as start, no endTime, and description <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
endSession (employee: User):<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** a currentlyActive session exists for this employee <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** set the active session's end to current time and currentlyActive to False <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
endSessionAt (employee: User, end: Time):<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** a currentlyActive session exists for this employee, end is after the session's start <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** set the active session's end to end and currentlyActive to False <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
cancelSession (employee: User):<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** a currentlyActive session exists for this employee <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** remove the active session for employee <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
removeSession (employee: User, session: Session):<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**requires** session exists and employee is the employee of this session <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
**effects** remove the session <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> **Notes**: There can only be one active session for each employee at a time to prevent conflicting time statements. The end Time parameter is optional until the session is no longer currentlyActive. The endSessionAt action specifically addresses the case where someone forgets to end a session. They can manually specify when the work actually ended. The cancelSession action handles cases where a session was started by mistake or the work was interrupted and shouldn't be recorded. The removeSession action is for cleaning up the database (say after a project is completed and fully recorded). It is assumed that this concept is queryable to find the needed information for record keeping.
