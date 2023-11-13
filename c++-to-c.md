# Quick introduction to porting C++ libraries to C
Most Arduino libraries are written in C++. Though this is not strictly a problem,
many people still prefer to develop embedded software in ordinary C.

The original library being written in C++ isn't a problem, however, as most C++
features used in embedded develompent map relatively well to standard C.

## Classes
A class can be quickly ported over to C by mimicking the way a class works.
This can be achieved by:
1. Defining a `struct` representing the internal state of the class. (self, this, etc.)
2. Defining instance methods as functions consuming the state `struct`
3. Implementing the private methods etc. using e.g. `static`, however this is not really necessary
4. Re-naming the methods when necessary, where a collision with the global namespace is likely

As an example, the following C++ header:
```cpp
class example_class
{
	public:
		example_class();
		void set(const unsigned long int val);
		unsigned long int get();
	private:
		unsigned long int _value;
}
```

Can be reimplemented as:
```c
typedef struct Example_class_state
{
	unsigned long int _value;
} Example_class_state

Example_class_state *example_class();

void set(Example_class_state *state, const unsigned long int value);
unsigned long int get(Example_class_state *state);
```

Additionally, the references to internal variables need to be changed
to use the state `struct` instead, as an example:
```cpp
void example_class::set(const unsigned long int value)
{
	_value = value;
}
```
to
```c
void set(Example_class_state *state, const unsigned long int value)
{
	state->_value = value;
}
```

As can be seen from the examples, the word `set` has a high probability
of being reserved by something else. We should take this into account when
naming the functions of our ported "class". As an example:
```c
void example_class_set(Example_class_state *state, const unsigned long int value);
```

## auto
`auto` keyword is not used in C, if it's being used somewhere in the original
code it has to be replaced by a proper static type. However, it is relatively
unlikely to encounter it in embedded software.

## I/O
Stream-IO should be relatively straightforward to port to C. Remember to
be careful with the formatting types, as they're automatically inferred
in the C++ stream IO implementation.
