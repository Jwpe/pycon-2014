# Porting Apps to Python 3

## Objectives

- 'straddling' Python 2/3
- Choosing target versions
- Porting as an iterative process
- Tests need to be GOOD before you can do this

## Porting Strategies

### Port once, abandon Python2

- Really hard
- 2to3 may be a useful starting point

### "Fix Up" at installation using 2to3

- 2to3 **painfully slow** on large codebases
- Bug reports don't match source

### 'Straddling' a single codebase

- Use "compatible subset" of Python
- Conditional imports to deal with changes to the standard library
- `six` module can help

## Targeting Python Versions
- <2.6 hard:
    - No `b''` literals
    - No `except Exception as e:`
    - Much more cruft and pain
    - 2.4 /2.5 are past EOL
- <3.2 hard:
    - PEP 3333 fixes WSGI
- 3.3 restores `u''` literals - good
- So go from 2.7 > 3.4

## Porting Risks

- Bugs can be introduced
- *fear* of breaking working software is the biggest barrier

### Managing porting risks

- Improve testing regime
- Use modernized idioms in Python2
- Be clear in text vs. bytes


### Dependencies

- Port packages with no dependencies first
- Then port packages with already-ported deps
    - Either find another package
    - Fork
    - Or submit PRs to fix it

## "Common Subset" idioms

- Lennart Regebro's book
- Modernize idioms in Python2
- `python2.7 -3` can point out problem areas
- Distinguish bytes vs text

### New syntax

- `print()`
- `except Exception as e`

## Testing

### Test Coverage
- 100% test coverage is ideal *before* porting
- For applications, functional testing is best
- Measure coverage

### Test Quality

- Work to improve *assertions*, not just coverage
- Doctests, meh. Hard to support

- For 2.x/3.x support, use Tox

## C Extensions

- Testing C is hard