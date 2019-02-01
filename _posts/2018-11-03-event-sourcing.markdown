---
title: "Event Sourcing - a case study"
layout: post
date: 2019-02-01 17:10
image: /assets/images/eventsourcing.png
headerImage: false
tag: 
- event sourcing
- node
- pattern
category: blog
description: "Event Sourcing pattern used in Elsevier"
author: LefterisTatakis
---
# Event sourcing

In this blog I would like to talk about a solution we implemented early 2018 which used Event Sourcing. I have been meaning in making this blog post since the day we deployed the solution to production, so here goes!

## What was the problem we were trying to solve

We were retrieving prices for our B2B platform were we needed to retrieve and update the pricing for our ebooks. These prices needed to be retrieved from two files and the data values would change on irregular intervals and we would not be notified of any changes. 

Therefore, we would need to daily poll the data source, to identify if any changes were made in the prices and then if so update our pricing.

The nature of the problem were were trying to solve was:

- Complicated data input structure
- Variety of pricing models within the data structure
- Multiple sources of data
- Only update the prices that change between executions of the application (if any)
- Auditing capabilities would be nice

The only requirement our side was that it would be TDD'ed (another great tool I was getting more hands on at the time).  

## Why are we using this pattern

I am sure other patterns and approaches may have been applicable for the problem we were trying to solve. However, this is what we felt met the criteria the business had set us and the technical benchmarks we had set ourselves. 

This pattern brought a three vital benefits with it:

- **Identification** of differences each time we have a new event
- **Accountability** and **traceability** of changes for the business. This makes it a great feature to rely on because every time the business asks us:
    - When did this product get added?
    - When did the price change?
    - Why was it removed?

    We have answers for all those questions within seconds by just looking at the events.

# What is Event Sourcing

Key entities within the pattern:

**Command** is a request made to the system to do something.

**State**  is the current form the data has.  The application reconstructs an entity’s ( a concept within our event store) current state by replaying the events.

**Event** is described as an action or change in the system, for example: a price change, a start of a new batch of prices or a product deletion. A key point about events is that they are immutable and can be stored using an append-only operation.

Finally, events should be defined in the Domain experts language in order to allow a representation of the system that matches the problem we are trying to solve. For example, types of Events we had were: 'PRICE_ADDED', 'PRICE_UPDATED', 'PRICE_DELETED'.

The interface we used for event was:

    export interface Event {
        name: string;
        id: number;
    }

where depending on the event type it was we would add extra attributes, eg prices.

**Projection** or materialised views are a generated current state of the system by replaying all the events in the system. We used this projection (table) to allow retrieval of the prices from other sources. 

Given these key entities we describe above, we have the following key components that bring the pattern together and manipulate these entities:

**State Retriever** replays all the events that have come before this point (if any) and generates the latest state of the system. 

**Command Generator** receives the input data and the current state of the system, to generate the changes that need to be made to the system.

**Command Handler:** given a specific type of command (a Handler) and the current `state` to generate the event that will trigger this change. For example, in our example we have the ImportHandler that will decide whether or not to add/change/remove a price from our model.

**Evolve:** The Evolver is the component that acts on the event through a series of Listeners. The listeners are classes that implements behaviours such as:

- Saving the event down to the event store
- Deleting prices from the projections
- Modifying the projection

# Our implementation of Event sourcing

<figcaption class="caption">An overview of our system</figcaption>
![Markdowm Image][1]

The key component of the application that puts this pattern together is here:
```javascript
    // Going to start add prices process 
    batchState = await this.stateRetriever.getCurrentBatchState();
    
    const batchEvent: BatchEvent = await this.batchHandler.handle({ name: 'START_BATCH' }, batchState);
    batchState = <BatchState> await this.batchEvolver.evolve(batchState, batchEvent);
    
    const commandGenerator = new CommandGenerator();
    const currentStates = await this.stateRetriever.getCurrentPriceState();
    
    const commands: ImportPriceCommand[] = commandGenerator.generate(prices, currentStates);
    
    for (const command of commands) {
    		// Check if current command has a current State. 
        let currentStateForISBN = ...
    		// if not instantiate 
    
    		// Generate appropriate event
        const importEvent: ImportEvent = await this.importHandler.handle(command, currentStates[currentStateForISBN]);
    
        if (!importEvent) {
            continue;
        }
        currentStates[currentStateForISBN] = await this.importEvolver.evolve(currentStates[currentStateForISBN], importEvent);
    }
    } catch (err) {
    	// error handling
    } finally {
    	const batchEvent: BatchEvent = await this.batchHandler.handle({ name: 'END_BATCH' }, batchState);
    	await this.batchEvolver.evolve(batchState, batchEvent);
    }
```
How do we get the latest state? From the `StateRetriever` of course!
```javascript
    public async getCurrentPriceState(): Promise<ImportState[]> {
            const allEvents: ImportEvent[] = await this.databaseManager.getAllImportEvents();
            const states: ImportState[] = [];
    
            for (const event of allEvents) {
                let currentState = states.find(state => state.isbn === event.isbn);
                if (!currentState) {
                    currentState = {
                        isbn: event.isbn,
                        lastEventId: 0,
                        accountPrices: {}
                    };
                    states.push(currentState);
                }
                currentState = await this.importEvolver.evolve(currentState, event);
            }
            return states;
        }
```
*Import Price Evolver* 
```javascript
    public async evolve (state: ImportState, event: ImportEvent): Promise<ImportState> {
            if (state.isbn !== event.isbn) {
                throw new Error('State and event isbns dont match!');
            }
    
            if ((state.lastEventId + 1) !== event.id) {
                throw new Error('State and event lastEventId are out of sync!');
            }
    
            state.accountPrices = this.squashPrices(event.prices, state.accountPrices);
            state.lastEventId = event.id; // Key point
    
            for (const listener of this.listeners) {
                await listener.notify(event);
            }
    
            return state;
        }
```
where `squarePrices` merges all the events and returns the latest state for that event.

In the line 
```javascript
    await listener.notify(event);
```
one scenario would be to save the event down into the event store (MySQL database), therefore, the listener of was really simple, a were more **Listeners**.
```javascript
    public async notify(event: ImportEvent): Promise<void> {
         await this.saveEvent(event);
    }
```
## The learning curve
<iframe src="https://giphy.com/embed/bEVKYB487Lqxy" width="480" height="264" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p></p>

Event sourcing was  a completely different mindset than any other solution I had ever approached.

The concept of encoding the changes in events, and then in order to retrieve the answer we would have to reply all the events each time, took me a while to understand. Especially when it came to the projection. 

What do you mean we have a current state of the data ( projection ) but in order to change the prices we don't pull the data from there but we will have to replay the data?

Once I was able to get that concept into my head... I think I was finally there :) 

There were a number of times I was thinking "wow we are over engineering this solution" and maybe we were. But the benefits of this solution and the learning experience of it were significant and invaluable.

The were a number of times I felt like this:
<iframe src="https://giphy.com/embed/NbgeJftsErO5q" width="480" height="380" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p></p>

## Benefits we are seeing now that we are using it

Through the nature of the pattern and the approach we took, TDD, the maintenance of this repo over the last year has been extremely straight forward and hassle free.

Reporting to the product owners and key business stakeholders has been straight forward.

## Conclusion

- Well worth the learning curve
- Reliable audit log
- Its one of the most maintainable features we have.
- It was a great learning experience.

Would recommend if it fits your problem :)

As always if there is something I have got wrong please let me know on <a href="https://twitter.com/LTatakis"> Twitter</a>.

[1]: /assets/images/eventsourcing.png