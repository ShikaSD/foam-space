# JS compose compiler plugin issues in 1.4

Current bugs that require working around with:
1. Mangler in JS checks for descriptor vs ir declaration signatures, that could be
  potentially disabled by a flag.
2. IR requires functions and classes to have the same signature as descriptors to be able to link them. I assume that could be fixed later.
 
 
- Potential problems here:
  - Compose changes signature of the functions - descriptors and ir functions will have different values. Therefore, frontend descriptors are not linked to deserialized IR correctly, and you have a bit of kerfuffle
    - **NOTE:** To fix that, I could probably leave stubs of functions used by frontend and then create "custom" functions to replace those.
  - At the same time, you cannot leave descriptors with old signatures, as compiler checkers will not let it through.
