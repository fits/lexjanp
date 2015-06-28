# Lexjanp (Lambda EXtensions Java ANnotation Processor)

Lexjanp is an experimental open source annotation processor for java.
It implements the feature like follows.

* do (Haskell)
* for (Scala)
* Computation Expression (F#)

## Feature

The annotation processor converts the special lambda that
```#{var}$do``` was an argument.

|before (source)|after|
|--|--|
|let #{lv} = #{rv}|#{var}.bind(#{rv}, #{lv} -> ・・・)|
|return #{expr}|#{var}.unit( #{expr} )|

### Example of conversion

##### before
```java
Optional<Integer> res = opt$do -> {
    let a = Optional.of(1);
    let b = Optional.of(2);
    return a + b;
```

##### after
```java
Optional<Integer> res = opt.bind(
    Optional.of(1),
    (a) -> opt.bind(
        Optional.of(2),
        (b) -> opt.unit(a + b)
    )
);
```


## Build

```
> gradlew build
```

## Sample

##### example/LaSample.java
```java
import java.util.function.Function;
import java.util.Optional;

public class LaSample {
    public static void main(String... args) {
        Opt opt = new Opt();

        Optional<Integer> o1 = Optional.of(2);

        Optional<Integer> res = opt$do -> {
            let a = o1;
            let b = Optional.of(3);
            let c = Optional.of(4);
            return a + b + c * 2;
        };

        // Optional[13]
        System.out.println(res);
    }

    static class Opt {
        public <T> Optional<T> bind(Optional<T> x, Function<T, Optional<T>> f) {
            return x.flatMap(f);
        }

        public <T> Optional<T> unit(T v) {
            return Optional.ofNullable(v);
        }
    }
}
```

##### Compile

```
> cd example
> javac cp ../build/libs/lexjanp-0.1.jar LaSample.java
```

##### Run
```
> java LaSample
Optional[13]
```
