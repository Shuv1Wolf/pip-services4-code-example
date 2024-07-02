```
import { MethodReflector } from "pip-services4-commons-node";

export class ClassA {
  public methodA(): number {
      return 123;
  }

  public methodB() {
      console.log("hello world b")
  }
}

export function main() {
  let myClassA = new ClassA();
  // Obtain all methods in classA
  
  let methods1 = MethodReflector.getMethodNames(myClassA);
  console.log("The methods in myClassA are: ", methods1);
  
  // Ask whether a specific method exists or not
  let methods2 = MethodReflector.hasMethod(myClassA, "methodA");
  console.log("methodA belongs to myClassA: ", methods2);
  
  let methods3 = MethodReflector.hasMethod(myClassA, "methodC");
  console.log("methodC belongs to myClassA: ", methods3);
  
  // Invoke a method in classA
  let methods4 = MethodReflector.invokeMethod(myClassA, "methodA");
  console.log("After running methodA the result is: ", methods4);
}

```

```
import { ObjectReader } from "pip-services4-commons-node";

export class ClassA {

  public param1: string = "hello";
  public param2: number = 123;

  public methodA(): number {
      return 123;
  }
}
export function main() {
  let myClassA = new ClassA();
  // Obtain all properties in ClassA
  
  let properties = ObjectReader.getPropertyNames(myClassA);
  console.log("The properties in myClassA are: ", properties);
  
  // Obtain the value of a property in classA
  let value1 = ObjectReader.getProperty(myClassA, "param1");
  console.log("The value of param1 is: ", value1);
  
  let value2 = ObjectReader.getProperties(myClassA);
  console.log("The properties and values in myClassA are: ", value2);
}
```

```
export function main() {
    // Obtain properties from a map(dictionary)
    let myMap = { 'key1': 123, 'key2': "ABC" };

    let hasProperty1 = ObjectReader.hasProperty(myMap, "key1");
    let value1 = ObjectReader.getProperty(myMap, "key1");
    console.log("MyMap contains key1: ", hasProperty1);
    console.log("The value of key1 is : ", value1);
  
    // Obtain properties from an array
    let myArray = [1, 2, 3];
    let hasProperty2 = ObjectReader.hasProperty(myArray, "5");
    let hasProperty3 = ObjectReader.hasProperty(myArray, "0");
    let value2 = ObjectReader.getProperty(myArray, "0");
  
    console.log("myArray contains an element with index 5: ", hasProperty2);
    console.log("myArray contains an element with index 0: ", hasProperty3);
    console.log("The value stored at postion 0 is: ", value2);
} 
```

```
import { ObjectReader, ObjectWriter } from "pip-services4-commons-node";

export class ClassA {

  public param1: string = "hello";
  public param2: number = 123;

  public methodA(): number {
      return 123;
  }
}

export function main(){
  let myClassA = new ClassA()

  let value1 = ObjectReader.getProperty(myClassA, "param1");
  console.log("The value of param1 is: ", value1);

  ObjectWriter.setProperty(myClassA, "param1", "hello 2");
  let value2 = ObjectReader.getProperty(myClassA, "param1");
  console.log("The new value of param1 is: ", value2);

  let myMap1 = { 'param1': 123, 'param2': "ABC" };
  ObjectWriter.setProperties(myClassA, myMap1);
  let value3 = ObjectReader.getProperties(myClassA);
  console.log("The new parameter values are: ", value3);

  // Map(dictionary)
  let myMap2 = { 'key1': 123, 'key2': "ABC" };
  ObjectWriter.setProperties(myMap2, { 'key1': 15422, 'key2': "ab" });
  let value4 = ObjectReader.getProperties(myMap2);
  console.log("The new values in the map are : ", value4);


  let myMap3 = { 'key1': 123, 'key2': "ABC" };
  ObjectWriter.setProperty(myMap3, "key1", "XYZ");
  value2 = ObjectReader.getProperty(myMap3, "key1");
  console.log("The new value in the map is : ", value2);

  // Array
  let myArray = [1, 2, 3];
  ObjectWriter.setProperty(myArray, "0", 123);
  value3 = ObjectReader.getProperty(myArray, "0");
  console.log("The new value in the array is : ", value3);
}
```

```
import { PropertyReflector } from "pip-services4-commons-node";

export class ClassA {

  public param1: string = "hello";
  public param2: number = 123;

  public methodA(): number {
      return 123;
  }
}

export function main(){
  let myClassA = new ClassA();

  // Obtain all property names
  let properties = PropertyReflector.getPropertyNames(myClassA);
  console.log("The properties of myClassA are: ", properties);

  // Find out whether an object has a property or not
  let has_param1 = PropertyReflector.hasProperty(myClassA, "param1");
  console.log("ClassA contains param1: ", has_param1);

  // Obtain all property names and their values
  let value3 = PropertyReflector.getProperties(myClassA);
  console.log("The properties of myClassA are: ", value3);

  // Change the value of a parameter
  let value1 = PropertyReflector.getProperty(myClassA, "param2");
  PropertyReflector.setProperty(myClassA, "param2", 14785);
  let value2 = PropertyReflector.getProperty(myClassA, "param2");
  console.log("The value of param2 is: ", value1);
  console.log("The new value of param2 is: ", value2);
}
```

```
import { RecursiveObjectReader } from "pip-services4-commons-node";

export class ClassAa {
  public param5: string = "hello aa";

}

export class ClassA extends ClassAa {

  public param1: string = "hello";
  public param2: number = 123;

  public methodA(): number {
      return 123;
  }
}

export class ClassB extends ClassA {
  public param4: string = "inside 2"; 
}

export function main(){
  let myClassA = new ClassA();
  let myClassB = new ClassB();

  let value1 = RecursiveObjectReader.getPropertyNames(myClassA)
  console.log("The property names of myClassA are: ", value1)

  let value2 = RecursiveObjectReader.hasProperty(myClassB, "param5")
  console.log("myClassB contains param5: ", value2)

  let value3 = RecursiveObjectReader.getProperties(myClassB)
  console.log("The properties of myClassB are: ", value3)

  let value4 = RecursiveObjectReader.getProperty(myClassB, "param4")
  console.log("The value of param4 is: ", value4)
}
```

```
import { RecursiveObjectReader, RecursiveObjectWriter } from "pip-services4-commons-node";

export class ClassAa {
  public param5: string = "hello aa";

}

export class ClassA extends ClassAa {

  public param1: string = "hello";
  public param2: number = 123;

  public methodA(): number {
      return 123;
  }
}

export class ClassB extends ClassA {
  public param4: string = "inside 2"; 
}

export function main(){
  let myClassB = new ClassB();
  let myClassC = new ClassB();

  // set_property
  RecursiveObjectWriter.setProperty(myClassB, "param2", "new value");
  let value1 = RecursiveObjectReader.getProperty(myClassB, "param2");
  console.log("The new values for the myClassB object are:", value1);

  // set_properties
  let myMap = { 'param1': 789456, 'param2': "ABCaccc" };
  RecursiveObjectWriter.setProperties(myClassB, myMap);
  let value2 = RecursiveObjectReader.getProperties(myClassB);
  console.log("The new values for the myClassB object are:", value2);

  // copy_proerties
  let value3 = RecursiveObjectReader.getProperties(myClassC);
  console.log("The properties of myClassC and their values are:", value3);
  RecursiveObjectWriter.copyProperties(myClassC, myClassB);
  let value4 = RecursiveObjectReader.getProperties(myClassC);
  console.log("The new properties of myClassC and their values are:", value4);
}
```

```
import { TypeDescriptor } from "pip-services4-commons-node";

export class ClassAa {
  public param5: string = "hello aa";

}

export class ClassA extends ClassAa {

  public param1: string = "hello";
  public param2: number = 123;

  public methodA(): number {
      return 123;
  }
}

export class ClassB extends ClassA {
  public param4: string = "inside 2"; 
}

export function main(){
  // Create type descriptors
  let type1 = new TypeDescriptor("ClassA", "library1");
  let type2 = new TypeDescriptor("ClassB", "library1");

  // equals
  let result1 = type1.equals(type2);
  console.log("type1 equals type2:", result1);

  // get_library
  let library1 = type1.getLibrary();
  console.log("The library of type1:", library1);

  // get_name
  let name1 = type1.getName();
  console.log("The name of type1 is:", name1);

  // from_string
  let typeDescriptor = TypeDescriptor.fromString("classA,library1");
  console.log("Type descriptor:", typeDescriptor);
}
```

```
import { TypeCode, TypeMatcher } from "pip-services4-commons-node";

export class ClassA {

  public param1: string = "hello";
  public param2: number = 123;

  public methodA(): number {
      return 123;
  }
}

export function main(){
  let objectA1 = new ClassA();

  // expected type: Object, actual type: classA, actualvalue: objectA1
  let type1 = TypeMatcher.matchType("Object", TypeCode.Object, objectA1);
  console.log("classA is an object:", type1);

  // expected type: Object, actual type: String
  let type2 = TypeMatcher.matchTypeByName("Object", TypeCode.String);
  console.log("String is an object:", type2);

  // expected type: classA, expected value: objectA1
  let type3 = TypeMatcher.matchValueType(typeof(objectA1), objectA1);
  console.log("objectA1 is of type classA:", type3);

  // expected type: Object, actual value: objectA1
  let type4 = TypeMatcher.matchValueTypeByName("Object", objectA1);
  console.log("ObjectA1 is of type Object:", type4);

  let string1 = "Hello World";
  let type5 = TypeMatcher.matchValueTypeByName("String", string1);
  console.log("string1 is of type String:", type5);
}
```

```
import { TypeReflector } from "pip-services4-commons-node";

export class ClassA {

  public param1: string = "hello";
  public param2: number = 123;
}

export function main(){
  let myClassA = TypeReflector.createInstanceByType(ClassA);
  console.log("The values of param1 and param2 are", myClassA.param1, "and", myClassA.param2);
}
```
