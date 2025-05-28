# Elephant Carpaccio Kata

The Elephant Carpaccio Kata is an exercise created by Alistair Cockburn to practice breaking down large features into small, vertical slices that deliver incremental value. The name plays on the idea of slicing work very thin, like carpaccio.

## Objective

Practice slicing a large feature into the thinnest possible vertical slices that still deliver value, then implementing them one by one.

## Exercise Overview

**Scenario**: You're building a point-of-sale system that needs to calculate totals, apply taxes, and handle various pricing rules.

### Original Requirements

1. Calculate the total for a shopping cart containing multiple items with quantities
2. Apply appropriate taxes based on location
3. Handle various discount rules (bulk, coupons, etc.)
4. Support multiple currency conversions
5. Generate an itemized receipt

### Carpaccio Approach

Instead of implementing all these features at once, the kata challenges you to slice them into the thinnest possible vertical slices that still deliver some user value.

## Sample Slicing Approach

1. **Slice 0**: Hardcoded return of a single item with quantity 1 and no taxes
   - Input: "1 item at $10"
   - Output: "Total: $10"

2. **Slice 1**: Handle multiple quantities of the same item
   - Input: "3 items at $10"
   - Output: "Total: $30"

3. **Slice 2**: Handle multiple different items
   - Input: "2 items at $5, 3 items at $10"
   - Output: "Total: $40"

4. **Slice 3**: Add basic tax calculation
   - Input: "2 items at $5, 3 items at $10" with 10% tax
   - Output: "Subtotal: $40, Tax: $4, Total: $44"

5. **Slice 4**: Add location-based tax rules
6. **Slice 5**: Add bulk discount rules
7. **Slice 6**: Add coupon support
8. **Slice 7**: Add currency conversion
9. **Slice 8**: Format itemized receipt

## Benefits of the Exercise

- Practice incremental development
- Learn to identify minimal viable slices
- Experience continuous delivery of value
- Improve estimation by working with small units
- Build confidence through frequent small successes

## Variations

1. Try different slicing approaches - what's the absolute thinnest first slice?
2. Experiment with different requirement sets
3. Try it with different architectural constraints (microservices, serverless, etc.)
