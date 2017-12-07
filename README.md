# Productineering
## About
### Purpose
A simple, platform-agnostic guide to writing sustainable software.

### Target Audience
Beginners who are comfortable with writing working but fragile software, and looking out for better architecture that is universally repeatable. Undergraduates in software-engineering are ideal readers.

### Guiding Principles
* **Universal** - We elect *Golang* as our language of choice due to its simplicity and limited feature-set, but the principles in *Productineering* must be platform-agnostic and universally applicable.
* **Minimalist** - While generics and reactive programming are immensly powerful tools, we choose to leave them out as they are but features that can be self-taught. Productineering strives for brevity and fundamentality.
* **Illustrative**  - Code is a universal language, and explains itself best.

## Hygiene in code
### Consistency
The following *must* be consistent across a code base, but ideally so should everything:

* **Nomenclature and convention** - Always begin by referring to the language's [style-guide](https://github.com/golang/go/wiki/CodeReviewComments). Naming can be evaluated with simplicity and expressiveness of our *ubiquitous language* (more on this later). 
* **Indentation and whitespacing** - Across blocks and even operators, whitespacing must be consistent so as to save the subsequent readers of cognitive stress. `gofmt` in *Golang* will take care of this.
* **Interfacing and API** - Predictability of interface saves developers from double-checking basic syntax, especially when code isn't working
* **Structure and Architecture** - If the things we define in our code are born and used the same way, the code-base seems more familiar much faster.

### Versioning
Use version-control regardless of whether it is teamwork or going solo. The ability to revert enables you to experiment with confidence instead of hesitation. We elect *Git* as our VCS of choice due to the flexiblity and speed of its branching-model and universal acceptance as an industry standard.

* Commit as often as possible with descriptive messages.
* Keep branch-names predictable - **fix_** and **feature_**. Avoid multiple features/fixes in the same branch.
* Keep a stable `master` or `master + dev` combination that is merged into, and rebased with, as often as possible.
* Keep sensitive and local-preferences out of version control with the appropriate `.gitignore` file. Use [gitignore.io](https://gitignore.io/) to bootstrap an initial draft rapidly.
* Learn the powers and dangers associated with `git reset --hard`.
* Do *NOT* use *git submodules* - they are overly complex and limited in utility. When, mostly for deployment purposes, we _do_ need to nest one repository in another, we use [Braid](https://github.com/cristibalan/braid) for vendoring.


### Documentation
Good code requires no documentation, but includes it nonetheless. For future reference, when this guide refers to *programming* it is assumed that documentation is included. While a strong ubiquitous-language (described later) will make most documentation seem extraneous, *Godocs* (in our case) are still tremendously useful. Ensure every publicly exported `interface`, `struct`, and `func` has a *Godoc*, as the linter is sure to remind you.

### Unit-tests
Setting up Unit-tests takes minimal investment but allow us to test individual modules in isolation. This is indispensable in locating bugs. When written with foresight, they can capture implicit expectations explicitly - for _eg._ the `type PostsStorage interface` may only guarantee us a `func FindAllPosts(user User) []Post`, but we have no assurance that:

1. Each `Post`'s `author User` field is the same as the passed `user User`
1. and for an unknown `user User`, we're returned a non-null, empty `[]Post{}` array (as opposed to just `nil`)

in *all implementations of interface `PostsStorage`* unless there's a unit-test for the same. In the future, you may swap `MySQLPostsStorage` for `MongoPostsStorage`, but will be reminded by your unit-tests when the implicit assumptions aren't translated appropriately. 

### Building
Setting up the code-base, and generating executable versions of a project must be as standardized, predictable, and automated as possible. This enables us to iterate rapidly. Since we're limiting this guide to *Golang*, we recommend using *Docker* to deploy your services. This enables you to describe infrastructure as code, and thereby version it. We then use *Kubernetes* to provision these *Docker* containers in the cloud.

We locally store all our code in `~/go/src/package-name` as is recommended for *Golang* - without this, tools such as `gorename` fail to work as expected.

### Automated testing
End-to-end tests help spot regression as well as benchmark any newly introduced slow-downs. We recommend *Behavioural Driven Design (BDD) testing*, with [Gingko](https://github.com/onsi/ginkgo). Keep end-to-end tests in the same repository as your code, since tools that automatically run tests when new code is merged typically require this.

## Planning software development
Any good developer spends ***less than 30% of her time programming***, having spent 50% planning, and 10% testing. The best software teams understand and influence product-design, and therefore the business itself. This can be achieved if the team's ability to craft *functional* and *technical* specifications is allowed to mature. The popularisation of this process may be accredited to [Joel Spolsky's commentary on specifications](https://www.joelonsoftware.com/2000/10/02/painless-functional-specifications-part-1-why-bother/).


### Functional Specifications
Function Specifications are group-discussions between creatives and editors - the document that arises out of these dicussions is only their outcome. The discussion concludes when the creative concurs with her editor upon the following with the simplest, briefest possible answers:

1. What is the problem this feature/fix attempts to solve?
2. Who are we solving it for, and what is the user-flow like?
3. Why is *this* the best solution to the problem?
4. *(Bonus)* What can we do *really* well to delight our user?

Each of the aforementioned questions aims at ensuring the developer can:

1. Challenge product-design choices.
2. Understand the impact of this _story_ - the feature is hereby referred to as a _story_ of how the product let the User do something, *eg. User can edit her questions with corrections or additional information*, as opposed to *Editable posts*.
3. Visualize actual usage, and relevancy to the customer that pays the software team's bills.

In addition to the above, the editor must internally reflect upon an additional question - *How does this feature fit into my product-market strategy?*

### Technical Specifications 
Technical Specifications are to Software Engineering, what Blueprints are to Civil Engineering. They are a translation of our Functional Specifications into actual programmatic requirements. A tech-spec must include the following:

#### Ubiquitous Language
The first step to solving  a problem, is the ability to express its complexity in an unambiguous language. *Objects*, *Structures*, *Variables*, *Functions*, and moreover the interactions among them, all require names. The business-team and engineering must talk the same parlance. 

Ubiquitous language is the first step towards architecting software. In fact, the _easiest_ way to evaluate architectural choices is to see if there's a simpler, more expressive language that describes it. [This blog-post](https://blog.carbonfive.com/2016/10/04/ubiquitous-language-the-joy-of-naming/) does a good job of demonstrating this exercise, but the reader is highly encouraged to look at as many examples as possible to fully grasp the impact of nomenclature.

Good nomenclature achieves:

* A beautiful separation of responsibilities among software modules
	* Enhances readability
	* Promotes reuse and modularity
	* Simplifies debugging
	* Makes the system resilient to change and iteration
* A natural API
	* Dramatically shortens time taken to integrate and deploy the system end-to-end
* Unambiguous and flowing communication within the team - what you say is what you mean

#### Proposed API
API is not only the web-service's exposed endpoints but also the method-signatures associated with internal `interface`s and `struct`s. As regards web-services, we usually opt for JSON-based RESTful [HTTP](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.5) API which has well-understood [best-practices](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api). Usually, the right ubiquitous language lends itself to API beautifully. Sometimes, considerations for efficiency require us to alter the conventions. Technical Specifications are the right place to document this as it enables other developers to build around this API like a black-box. This helps avoid haphazard last-minute changes while integrating the many moving parts of the software together.

#### Time Estimate
Once the Ubiquitous Language and Proposed API are in place, a developer is in a position to propose a detailed checklist of tasks to complete the story. Insist on further decomposing every single item on the checklist, until it can no longer be expanded. At the most granular level, tasks should ideally require no more than 15-45 minutes of programming time.

Adding these micro-estimates gives us a story-completion timeline. We use a buffer of 50% (or 1.5x of the original estimate), since software development is never without its vagaries.

## Layers of software
### Application
The Application is the user-facing interface (graphical or command-line), or as is the case of server-ware, RESTful API.
### Business
Below the Application lies our Business layer. This consists of Business logic and rules.
### Domain
The Domain consists of all our Value Objects and Entities from the Ubiquitous Language for the Business layer to process upon.
### Persistence & External Services
The persistence layer consists of code that connects to databases and other third-party services, including the kernel itself.
### Inter-layer communications
When a layer above wants to talk to a layer below, it uses it's API and calls one of it's methods. When a layer below wishes to relay information to a layer above, it does so via *callback*.

## Developing our domain
### Immutabile Values, Entities and Factories
There has been much hype about Object-Oriented Programming in recent times, although modern type-oriented languages are just as rich (and as we'll soon find, safer). In fact, with the stark exception of generics, the *Golang* approach to any problem will almost always be more sustainable than in *Java*, and because of its type-oriented nature.

Take for example the following Java snippet:

```java
class User {
	Money money;
	
	User(Money money) {
		this.money = money;
	}
}

class Money {
    private double money;
    
    Money(double money) {
    	this.money = money;
    }
    
    void add(Money moreMoney) {
    	this.money += moreMoney.money;
    }
    
    // More useful methods...
}

// Init our users
Money m = new Money(100.0);
User me = new User(m);

m.add(50.0);
User you = new User(m);
```

The discerning reader may have already spotted the so-called *side-effect* - we intended to give `me` and `you` a sum of `$100` and `$150` respectively, but due to the shared object-reference, have incorrectly assigned `$150` to `me` as well.

If, however, `Money` behaved just like any other primitive-type, and was passed-by-value instead of reference, we could eliminate the chance of such mistakes from our code altogether.

```go
type User struct {
	money Money
}

type Money struct {
	money float64
}

// Add must return Money, since `m` is local-only.
func (m Money) Add(moreMoney Money) Money {
	m.money += moreMoney.money
	return m
}

func Main() {
	money := Money{100}
	me := User{money}
	moreMoney := money.Add(50)
	you := User{moreMoney}
}
```

This example is far more readable and explicit in its intentions. Can this be done in Java? Yes - our `add()` method would look something like this:

```java
Money add(Money moreMoney) {
	Money ret = new Money(this.money); // This wouldn't scale as Money becomes more complex
	ret += moreMoney.money;
	return ret;
}
```
This *clone-this* approach becomes cumbersome as we add more fields to `Money`. Type-oriented languages such as *Go* do this automatically for us by supplying us a cloned `m` to begin with.

#### Isn't cloning `struct`s expensive?
Modern-day compilers are smart enough to know when to clone (upon write), and when to share the reference (read-only) - *ie.* cloning happens *lazily*.

#### Isn't `Money` just a glorified `float64`?
Wrapping primitives with named `struct`s achieves more than just *logical grouping of primitives* for us - it ensures *validity* if used appropriately.

```go
// NewMoney is a Factory for Money.
func NewMoney(money float64) (Money, error) {
	if money >= 0 {
		return Money{money}, nil
	}
	
	return Money{}, errors.New("money can't be negative")
}
```

By introducing a convention of constructing `Money` only via the `NewMoney` *Factory* function, we've reduced the set of possible values `Money` can take to non-negative integers alone. This means one less `if-else` everywhere else in our business logic dealing with money.

In case of complex factories (typically when construction is a stateful operation), one can also use `struct`s for Factories. This has the advantages of composition and injectability (as we'll see later in this guide), but at the cost of being verbose.

```golang
type MoneyFactory interface {
	Make() (Money, error)
}

// Imagine a method on our App
func (app App) DisplayMoney(f MoneyFactory) {
	fmt.Printf("%f\n", f.Make().money)
}

type FloatMoneyFactory struct {
	amount float64
}

func (f FloatMoneyFactory) Amount(amount float64) MoneyFactory {
	f.amount = amount
	return f
}

func (f FloatMoneyFactory) Make() (Money, error) {
	if f.amount >= 0 {
		return Money{f.amount}, nil
	}
	
	return Money{}, errors.New("money can't be negative")
}

// FloatMoneyFactory is a MoneyFactory by duck-typing, and 
// can be passed around into Applications like so...
app.DisplayMoney(FloatMoneyFactory{}.Amount(100)) // Note that DisplayMoney takes any MoneyFactory
```
Here, `app.DisplayMoney(MySQLMoneyFactory{}.User("johndoe"))` would work just as well, enabling `DisplayMoney(f MoneyFactory)` to be completely agnostic to where the `Money` actually comes from.

#### So what *is* `Money`?
`Money` is a *Value*, or something that is equatable by its value alone. `User` on the other hand, is an *Entity*, or something which may have its values change in time, but will be equatable by its identity, such as say `User#ID int`. *Values* and *Entities* are the building-blocks of our *Domain* layer.

### Repositories 
The business layer for most useful applications requires us to persist state. Since data in the business layer is represented in *Values* and *Entities*, we need an interface between the *Domain* layer and our *Persistence* technology. These are popularly referred to as *Repositories*, and look something like this:

```go
type UserReposity interface {
	Save(user User) error
	FindByID(id int) (User, error)
}

// InMemoryUserRepository implements UserRepository.
type InMemoryUserRepository struct {
	users []User
}

func NewInMemoryUserRepository() InMemoryUserRepository {
	return InMemoryUserRepository{make([]user, 0)}
}

func (r *InMemoryUserRepository) Save(user User) error {
	for i, u := range r.users {
		if user.Equals(u) {
			r.users[i] = user
			return nil
		}
	}
	
	r.users = append(r.users, user)
	return nil
}

func (r *InMemoryUserRepository) FindByID(id int) (User, error) {
	for _, ret := range r.users {
		if ret.id == id {
			return ret, nil
		}
	}
	
	return User{}, errors.New("user by that ID doesn't exist")
}
```

Business logic now only requires a `UserRepository` and is agnostic to whether that `UserRepository` is actually a `InMemoryUserRepository`, or a `MySQLUserRepository`. This also makes it easier to mock our database for test-cases.

#### A note on ORMs
ORMs intend on giving us a combination of domain-layer objects that also include persistence-logic for themselves. While they do offer syntactic-sugar and convenience, experienced developers steer clear of ORMs because of the lock-in with one singular ORM due to this *spillover* of persistence into the domain, and the automagic (*eg.* incremented integer IDs, or timestamps) that is often out of the developer's control.

#### Designing a Repository interface
The methods that are necessitated in a Repository's interface are determined by the *Applications* required in our business layer. For instance, in the aforementioned example, we probably had a `GET /users/<ID>` application/route in a server, that required us to guarantee `UserRepository#FindByID(id int)` in the interface.

Oftentimes, one may find *Applications* requiring a combination of two or more entities, that are illegal if they exist independent of each other. For instance, a `UserProject` allows us to authenticate a `User` for a particular `Project` in our RESTful APIs. These are known as *Aggregate Roots*. Of course, `User` by itself will be an aggregate root for *Applications* that update the User profile. It is the identification of these aggregates that in fact, define our repository interfaces.

### Specifications
Repositories successfully separate persistence technology from business rules. However, certain domain-layer rules often become implicit in the implementation of the Repository interface. Take for example, the fact that all `Project`s must contain at least one `User` in its `owners` field. This is a domain-level constraint, that is now checked inside the persistence-code of an implementation of the Repository.

Specifications are a pattern to make domain-level relationships explicit, so that changes in the persistence-layer are guaranteed to enforce them.

```go
type ProjectOwnershipSpecification struct {
}

func (s ProjectOwnershipSpecification) Ensure(project Project) error {
	if len(project.owners) > 0 {
		return errors.New("project must contain at least one owner")
	}
	
	return nil
}

type ProjectRepository struct {
	delegate IProjectRepository
	ownership ProjectOwnershipSpecification
}

type IProjectRepository interface {
	Save(project Project) error
	FindByID(id int) (Project, error)
}

func (r ProjectRepository) Save(project Project) error {
	if err := r.ownership.Ensure(project); err != nil {
		return err
	}
	
	return r.delegate.Save(r)
}

func (r ProjectRepository) FindByID(id int) (Project, error) {
	var (
		ret Project
		err error
	)
	
	if ret, err = r.delegate.FindByID(id); err != nil {
		return ret, err
	}
	
	err = r.ownership.Ensure(ret)
	return ret, err
}
```

With `delegate`, you can inject any underlying persistence technology, and at the same time ensure the integrity of data before returning it to the business layer.

### Services
Services are the generalised-case of Repositories, separating the business layer from *any* underlying layers (*ie.* not just the persistence layer). Services are handy when one wants to communicate over network to third-party or another internal services. 

```go
type SMSService interface {
	Send(to Phone, body string)
}

// Can be implemented by TwilioSMSService, PlivoSMSService, etc.
```

### Intents/Applications and Dependency Injections
Once we've built Repositories and Services, all that remains is to tie them together to perform real-world tasks in the business layer. This happens in *Applications*, or as we call them - *Intents*.

```go
type ViewPostIntent struct {
	// Dependencies
	repo PostRepository
}

func (intent ViewPostIntent) Enact(id int) {
		if post, err := intent.repo.FindByID(id); err == nil {
			// Display the post 
		}
}
```

Notice that `PostRepository` is an interface, and can be implemented by any persistence technology - SQL, NoSQL, in-memory, etc. It is said to be *constructor-injected* into *ViewPostIntent* during the latter's construction; as simple as `ViewPostIntent{MySQLPostRepository{}}`. This is the crux of *dependency-injection* - externally inject dependencies instead of constructing them inside the dependent. This way, the dependent's source-code no longer hard-codes the exact implementation:

```go
type ViewPostIntent struct {
}

func (intent ViewPostIntent) Enact(id int) {
	repo := MySQLPostRepository{} // MySQL is hard-coded if not injected - this is a problem! ⚠️
	// ...
}
```

Dependencies in turn often contain their own set of dependencies, and as a result we're left with a graph of dependencies - all stemming out of one single root `struct`. As you can imagine, this graph is extremely painful to construct manually and hence, a dependency-injection *container* or framework, is necessitated. We recommend you start off with simple constructor-injections and move to advanced techniques only when the object-graph becomes unmanageable manually.

### Composition over Inheritance
Often, some objects/structures share certain traits and logic. Take for instance `AuthenticateUserIntent` - required in every other intent to authenticate the request. We do this with *composition*.

```go
type ViewPostIntent {
	// Mixins
	AuthenticateUserIntent
	
	// Dependencies
	repo PostRepository
}

func (intent ViewPostIntent) Enact(secret string, id int) {
	if err := intent.Auth(secret); err == nil { // Auth(string) is a method of AuthenticateUserIntent
		// ...
	}
}
```

If we were to do this in Java, we would have naturally gravitated towards inheritance, with `ViewPostIntent inherits AuthenticateUserIntent`. However, bear in mind that one can `inherit` only a single parent. If we were to introduce another handy mixin, the `GoogleAnalyticsIntent`, in all our authenticated Intents, we'd be looking at unnatural inheritances just to fit into the single-parent constraint.

#### Composition versus inheritance
Multiple inheritence is semantically impossible because each parent can contain its own private state, and how that interacts with other parents is undefined behaviour. In the case of composition, we're essentially embedding the multiple 'parents' within the root and simply aliasing method calls on the root to the parent they belong to. This allows our composition to be stateful.

In the *Swift3* programming language, composition is compulsorily stateless - protocol extensions only define methods, not stored properties. While protocol extensions to allow us extend classes in very interesting ways, the *Golang* composition is a simple treatment to the problem.