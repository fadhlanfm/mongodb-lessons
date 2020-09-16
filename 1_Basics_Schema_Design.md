# Basic Schema Design

## One-to-Few

    > db.person.findOne()
    {
        name: 'Kate Monster',
        ssn: '123-456-7890',
        addresses : [
    	    { street: '123 Sesame St', city: 'Anytown', cc: 'USA' },
    	    { street: '123 Avenue Q', city: 'New York', cc: 'USA' }
        ]
     }

## One-to-Many

Each Part would have its own document:

    > db.parts.findOne()
    {
      _id : ObjectID('AAAA'),
      partno : '123-aff-456',
      name : '#4 grommet',
      qty: 94,
      cost: 0.94,
      price: 3.99

Each Product would have its own document, which would contain an array of ObjectID references to the Parts that make up that Product:

    > db.products.findOne()
    {
      name : 'left-handed smoke shifter',
      manufacturer : 'Acme Corp',
      catalog_number: 1234,
      parts : [ // array of references to Part documents
        ObjectID('AAAA'), // reference to the #4 grommet above
        ObjectID('F17C'), // reference to a different Part
        ObjectID('D2AA'),
        // etc
      ]

You would then use an **application-level join** to retrieve the parts for a particular product:

        // Fetch the Product document identified by this catalog number
    > product = db.products.findOne({catalog_number: 1234});
        // Fetch all the Parts that are linked to this Product
    > product_parts = db.parts.find({_id: { $in : product.parts } } ).toArray() ;

As an added bonus, this schema lets you have individual Parts used by multiple Products, so your One-to-N schema just became an N-to-N schema without any need for a join table!

## Basics: One-to-Squillions

This is the classic use case for “parent-referencing” – you’d have a document for the host, and then store the ObjectID of the host in the documents for the log messages.

    > db.hosts.findOne()
    {
      _id : ObjectID('AAAB'),
      name : 'goofy.example.com',
      ipaddr : '127.66.66.66'
    }
    > db.logmsg.findOne()
    {
    	time : ISODate("2014-03-28T09:42:41.382Z"),
    	message : 'cpu is on fire!',
    	host: ObjectID('AAAB') // Reference to the Host document
    }

```
> db.logmsg.findOne()
  {
    time : ISODate("2014-03-28T09:42:41.382Z"),
    message : 'cpu is on fire!',
    host: ObjectID('AAAB') // Reference to the Host document
  }
```

You’d use a (slightly different) application-level join to find the most recent 5,000 messages for a host:

      // find the parent ‘host’ document
    > host = db.hosts.findOne({ipaddr : '127.66.66.66'}); // assumes unique index
      // find the most recent 5000 log message documents linked to that host
    > last_5k_msg = db.logmsg.find({host: host._id}).sort({time : -1}).limit(5000).toArray()

## Recap

So, even at this basic level, there is more to think about when designing a MongoDB schema than when designing a comparable relational schema. You need to consider two factors:

- Will the entities on the “N” side of the One-to-N ever need to stand alone?

- What is the cardinality of the relationship: is it one-to-few; one-to-many; or one-to-squillions?

Based on these factors, you can pick one of the three basic One-to-N schema designs:

- Embed the N side if the cardinality is one-to-few and there is no need to access the embedded object outside the context of the parent object

- Use an array of references to the N-side objects if the cardinality is one-to-many or if the N-side objects should stand alone for any reasons

- Use a reference to the One-side in the N-side objects if the cardinality is one-to-squillions

[Source](https://www.mongodb.com/blog/post/6-rules-of-thumb-for-mongodb-schema-design-part-1)
