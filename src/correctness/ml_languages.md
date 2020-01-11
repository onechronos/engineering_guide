# ML Languages - Our Secret Weapon

One of the key advantages of functional, strongly typed languages such as ocaml (ReasonML) is clear separation of data from functionality and the expressiveness of types when it comes to modeling said data and dependencies. Let's revisit the previous example by writing out the types that implicitly took part in ```is_short_sell_eligible```:

```reasonml
type instrumentID =
  | InstrumentID(int);

type orderSide =
  | Buy
  | Sell
  | ShortSell;

type order = {
  instrumentID,
  orderSide,
};
```

To read the above, four rules for reading the code form of nouns in the Ubiquitous Language will suffice:

1. Types represent the shape of data - they're nouns
2. Types consist of other types
3. Types that have ```|``` in them mean "one of these types"
4. Types enclosed in ```{ ... }``` means "all of these types"

Given that, and concrete examples of how to read type signatures, e.g.:

* ```instrumentID``` takes one form - an ```InstrumentID``` that contains an integer
* ```orderSide``` has three forms: ```Buy```, ```Sell```, ```ShortSell```; none of those forms contains anything else
* ```order``` contains an ```orderId``` and an ```orderSide```

With minimal background on how to read the language the business side could look at this code point out that ```ShortSellExempt``` is missing as an option for ```orderSide```, and that ```order``` should contain a field representing a locate.

Actioning this, the developer would then write:

```reasonml
type orderSide =
  | Buy
  | Sell
  | ShortSell
  | ShortSellExempt;

type locateRequired =
  | Yes
  | No;

type order = {
  instrumentID,
  orderSide,
  locateRequired: option(locateRequired),
};
```

and explain that there's a fifth rule: ```option``` means that something might or might not be present. At this point, the developer still hasn't written any business logic. The business surfaced the issue with the domain model before time went towards writing "doomed to fail" code.

Now that the business side and the developer have agreed on the shape of the data itself, it's time to agree on the verbs. Before writing the actual logic, the developer could sketch out function definitions for feedback:

```reasonml
let isSaleProperlyFlagged: order => bool;
let isSecurityShortRestricted: order => bool;
let isShortSellEligible: order => bool;

let isShortSellEligible = order =>
  isSaleProperlyFlagged(order) &&
  isSecurityShortRestricted(order);
```

A quick lesson on reading these type signatures would make it clear what's happening, e.g. ```isSaleProperlyFlagged``` is a function that takes an ```order``` as an input and returns ```true/false```. At this point, the business should question the ambiguity of what happens when ```isShortSellEligible``` receives a buy order. That would lead to the correction:

```reasonml
let isSell: order => bool;
let isShortSellEligible = order =>
  isSell(order) &&
  isSaleProperlyFlagged(order) &&
  isSecurityShortRestricted(order);
```

With the shape of the validation nailed down the developer can write:

```reasonml
let isSell = order => order.orderSide != Buy;

let isLocateGood = order =>
  switch (order.orderSide, order.locateRequired) {
  | (ShortSell, Some(Yes)) => true
  | (ShortSell, _) => false
  | (Buy | Sell | ShortSellExempt, _) => true
  };

let isSaleProperlyFlagged = order =>
  switch (order.orderSide, order.locateRequired) {
  | (Buy | Sell, _) => false
  | (ShortSell, _) => isLocateGood(order)
  | (ShortSellExempt, _) => true
  };

let isSecurityShortRestricted = order =>
  switch (order.orderId) {
  | OrderID(1234) => true
  | _ => false
  };
```

A quick explanation of pattern matching semantics can make ```switch``` statements almost as readable as types. Short of that the developer could walk a stakeholder through every match arm, explain the conditions that lead to it, and verify and ask "is the function returning the right value in this case?"

Both the process and result are superior to V1 of ```is_short_sell_eligible```. In contrast to the process that led to the business discovering issues with the validator after the developer spent time writing it, our functional/ML DDD process surfaced it *before*. That was immaterial in this trivial example, but early detection of modeling errors can often save weeks or months. Furthermore, every noun and verb came straight from the Ubiquitous Language - there's nothing that would appear foreign to either a technical or non-technical stakeholder.

The question becomes: how well does this approach scale to larger projects and non-trivial business logic? In short, exceptionally well. Real-world systems contain enormous complexity in the portions of the codebase that developers deal with, but human produced business logic typically looks like a slightly expanded version of ```isShortSellEligible```. Business logic can itself be large and complex, but it's typically decomposable using the same incremental approach that we used to arrive at the final version of ```isShortSellEligible```.

Decomposition and method extraction often results in code that cleanly separates the implementation details from the pure functional domain that makes up actual business logic. The module systems typical of ML languages allow for a clean separation between types, method signatures (verbs), and the concrete implementations of both. As such, it's possible to create a separate "repository" of pure business logic in which all stakeholders collaborate with high fidelity and minimal overhead.

We won't go into the other technical merits of the DDD functional approach here, but there are plenty.
