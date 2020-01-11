# Preventing Type II/III Faults

One core tenant of DDD is the separation of business logic from "plumbing and boilerplate." Business logic is code of the form:

```python
def is_short_sell_eligible(market_context, order):
    if market_context[order.instrument_id].is_short_restricted:
        return False

    if order.side == OrderSide.SELL_SHORT:
        return True

    return False
```

Plumbing and boilerplate is code of the form:

```python
    class MarketContext:
        def __getattr__(self, instrument_id):
            if not isinstance(instrument_id, InstrumentID):
                raise ValueError
            ...etc
```

Ideally, stakeholders should be able to see and read business logic code without being programmers themselves. In this specific example, no one except for the development team implementing it needs to care about how ```MarketContext``` works internally. Stakeholders on the business side can read the code for ```is_short_sell_eligible``` and understand it after receiving the explanation that

```python
if market_context[order.instrument_id].is_short_restricted
```

means: "look up the current (broader market) conditions for the symbol corresponding to ```order``` and return ```true``` if the security is short restricted."

If we had mixed business logic with implementation we might have ended up with something like:

```python
def is_short_sell_eligible(order):
    try:
        ord_restriction_status = conn_pool.take(). \
            execute("SELECT * FROM tbl_mkt_status WHERE " \
                    "inst_id == {}", order.instrument_id)
        if not ord_restriction_status == 0:
            return False
    except DBConnectionError as db_conn_error:
        raise Execption("...")
    except Exception:
        raise

    if order.side == 5:
        return True

    return False
```

Reading this and understanding the core logic independent of the plumbing is a high bar, even for developers. As it turns out, both the original version and the co-mingled version contain two logical errors. Neither checks that the order had a valid locate, and neither accounted for the fact that ```OrderSide.SHORT_EXEMPT = 6``` *could* result in an order that's short eligible.

No amount of testing or even the use of formal methods would catch this. The programmer's mental model of the algorithm ```is_short_sell_eligible``` was wrong, so any tests or proofs of correctness implemented to verify it would instead verify that the *wrong* model was *wrong in the right way.*

DDD is not the lone approach when it comes to dealing with this problem. Every software development methodology imposes its own views on how to gather requirements and action them. DDD is notable in its emphasis on modeling the problem domain in a way that for business-readable code as opposed to a business readable model that's subsequently translated to code. This, and the use of a Ubiquitous Language that's understood by technical and non-technical stakeholders reduces the probability of requirements getting lost in translation.
