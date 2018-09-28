Article FP domain design
- Separate data from behaviors
- ADTs in model package
- Typeclasses for common reusable behaviors
- algebras in services / repository package
  - trait parametrised
  - encode effects
  - separate business logic (in algebras) from execution effects (in interpreters)
- implementations in sub modules impl
  - trait concret + object ?
- avoid implementation from leaking to client of algebras
  - tagless final
- commit to implementation and concrete type at the latest moment
  - tagless final
- handle effects stack
  - mtl ? monad transformer ? natural transformation ?


