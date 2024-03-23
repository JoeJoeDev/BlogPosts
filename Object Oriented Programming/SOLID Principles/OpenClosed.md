Open Closed Principle in C#
===========================

Software entities should be open for extension and closed for modification. An entity should allow its behavior to be extended without the need to modify its source code.

If we change the source code of an existing bug-free entity in order to support a change in requirements, we run the risk of introducing bugs into what was previously a bug-free entity.

What the Open Closed Principle is trying to enforce is the ability to keep our entities source code the same, by closing it to modifications but allowing its functionality to be extended by implementing interfaces and abstract classes.

It can be hard to know when to use an abstract class and when to use an interface – An abstract class allows you to provide functionality to the classes that extend the abstract class, whereas an interface does not provide any functionality but is more of a contract that defines methods and properties that a class is required to implement.

Another thing to note is that in some languages, a class can only extend one other class but can implement many interfaces.

As always the best way to show how to implement anything is with some sample code

    ```
        public class Portion 
        {
            public decimal Size { get; set; }
        }
        
        public class Cat 
        {
            private Portion portion;
    
            public Cat(string name)
            {
                this.Name = name;
                this.portion = new Portion {Size = 5 };
            }
    
            public string Name { get; }
    
            public string EatFood()
            {
                return $"Meow, Meow, {this.Name} ate {this.portion.Size} cat biscuits";
            }
        }
    
        public class Dog
        {
            private Portion portion;
    
            public Dog(string name)
            {
                this.Name = name;
                this.portion = new Portion { Size = 10 };
            }
    
            public string Name { get; }
    
            public string EatFood()
            {
                return $"Woof, Woof, {this.Name} ate {this.portion.Size} dog biscuits";
            }
        }
    
        public class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Feeding time!");
                List<dynamic> animals = new List<dynamic> {
                    new Cat("KiKi"),
                    new Dog("Rover")
                };
    
                foreach (var animal in animals)
                {
                    switch (animal.GetType().Name.ToLower())
                    {
                        case "dog":
                            {
                                Console.WriteLine(((Dog)animal).EatFood());
                                break;
                            } 
                        case "cat":
                            {
                                Console.WriteLine(((Cat)animal).EatFood());
                                break;
                            }
                    }
                }
    
                Console.ReadKey();
            }
        }
    ```

In this example, we have a Cat and Dog class which are being fed a portion of food. The first problem is that if we want to change the amount of food each Cat or Dog is being fed we need to change their class, which is a violation of the Open Closed Principle, and the fact that they need to worry about how much they are being fed is a violation of the Single Responsibility Principle, plus I don’t really think that a Cat or a Dog would make a healthy decision about portion size, so let’s change that first.

    
        public class Cat 
        {
            private Portion portion;
    
            public Cat(string name, Portion portion)
            {
                this.Name = name;
                this.portion = portion;
            }
    
            public string Name { get; }
    
            public string EatFood()
            {
                return $"Meow, Meow, {this.Name} ate {this.portion.Size} cat biscuits";
            }
        }
    
        public class Dog
        {
            private Portion portion;
    
            public Dog(string name, Portion portion)
            {
                this.Name = name;
                this.portion = portion;
            }
    
            public string Name { get; }
    
            public string EatFood()
            {
                return $"Woof, Woof, {this.Name} ate {this.portion.Size} dog biscuits";
            }
        }
    

Now we have moved the responsibility of portion size away from both the Dog and Cat classes meaning that we no longer need to change the Cat or Dog classes when a portion changes, but what if we need to give a Dog and Cat a different type of Portion for some reason? maybe there is a way of calculating the portion requirements based on a height and age?  
We can fix this by providing an abstraction of the Portion class in the form of an interface. Let’s do that now.

    
    public interface IPortion
        {
            decimal Size { get; }
        }
    
        public class SmallPortion : IPortion
        {
            public SmallPortion()
            {
                Size = 5;
            }
    
            public decimal Size { get; }
        }
    
        public class LargePortion : IPortion
        {
            public LargePortion()
            {
                Size = 10;
            }
    
            public decimal Size { get; }
        }
    
        public class Cat
        {
            private IPortion portion;
    
            public Cat(string name, IPortion portion)
            {
                this.Name = name;
                this.portion = portion;
            }
    
            public string Name { get; }
    
            public string EatFood()
            {
                return $"Meow, Meow, {this.Name} ate {this.portion.Size} cat biscuits";
            }
        }
    
        public class Dog
        {
            private IPortion portion;
    
            public Dog(string name, IPortion portion)
            {
                this.Name = name;
                this.portion = portion;
            }
    
            public string Name { get; }
    
            public string EatFood()
            {
                return $"Woof, Woof, {this.Name} ate {this.portion.Size} dog biscuits";
            }
        }
    

Great! now we can change the type of portion we give to the Dog and Cat classes, as long as the new Portion class implements the IPortion interface, we can feed them any portion we want without the need to change either the Cat or Dog class.

Now I think it’s time to have another look at the Program class

    
        public class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Feeding time!");
                List animals = new List {
                    new Cat("KiKi", new SmallPortion()),
                    new Dog("Rover", new LargePortion())
                };
    
                foreach (var animal in animals)
                {
                    switch (animal.GetType().Name.ToLower())
                    {
                        case "dog":
                            {
                                Console.WriteLine(((Dog)animal).EatFood());
                                break;
                            }
                        case "cat":
                            {
                                Console.WriteLine(((Cat)animal).EatFood());
                                break;
                            }
                    }
                }
    
                Console.ReadKey();
            }
        }
    

The Program class is doing too much, it should not need to know how to feed the animals, this is a violation of the Single Responsibility Principle. We have the horrible switch statement that is doing a type check so we can cast to the correct object type to call the EatFood method, which we will need to add a new condition to every time we want to support a new animal, which violates Open Closed Principle.

Let start by adding a new interface that our Cat and Dog classes can implement,

    
        public interface IEat
        {
            string EatFood();
        }
    
        public class Cat : IEat
        {
        .....
    
        public class Dog : IEat
        {
        .....
    
    

As you can see, both the Cat and Dog class already have the EatFood method implemented, so we don’t have any extra work to do on them. We can now change the type of our animals list within the Program classes Main method from “dynamic” to “IEat” like so.

    
        List<IEat> animals = new List<IEat> {
            new Cat("KiKi", new CatPortion()),
            new Dog("Rover", new DogPortion())
        };
    
    

Now let’s do something about that switch statement. We will create a new class called AnimalFeeder which we will initialise with our animal list that can take care of feeding the Cat and Dog instances, as well as any other classes that implement the IEat interface.

    
        public class AnimalFeeder
        {
            private List<IEat> animals;
    
            public AnimalFeeder(List<IEat> animals)
            {
                this.animals = animals;
            }
    
            public void Feed()
            {
                foreach (var animal in animals)
                {
                    Console.WriteLine(animal.EatFood());
                }
            }
        }
    

We have moved the loop and switch statement out of the Main method of the Program class and placed it within the Feed method of our new AnimalFeeder class, but we no longer have to deal with the switch statement or the typecasting as our animal classes all implement the IEat interface which enforces the implementation of the EatFood method on each class, allowing us to call EatFood() on each instance within our animals list.

Our Program class now looks like this

    
        public class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Feeding time!");
                List<IEat> animals = new List<IEat> {
                    new Cat("KiKi", new SmallPortion()),
                    new Dog("Rover", new LargePortion())
                };
    
                new AnimalFeeder(animals).Feed();
                Console.ReadKey();
            }
        }
    

Now adding a new animal class is super easy. We just add another animal class, implement the IEat interface on the new class and add a new instance of our class to the list of animals we are passing to the AnimalFeeder.

    
        public class Rat : IEat
        {
            public string EatFood()
            {
                return "Sniff, Sniff, This wild rat ate all the scraps in the kitchen";
            }
        }
    
        public class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Feeding time!");
                List<IEat> animals = new List<IEat> {
                    new Cat("KiKi", new SmallPortion()),
                    new Dog("Rover", new LargePortion()),
                    new Rat()
                };
    
                new AnimalFeeder(animals).Feed();
                Console.ReadKey();
            }
        }
    
    

We have made the application conform to the Open Closed Principle by making it extendable by using interfaces, which is allowing us to pass in new objects as the scope of the application changes without the need to change other classes. We no longer need to add a new type check to the switch statement every time a new class implements IEat and needs to be fed. We can pass in any type of Portion that we want to a Dog or a Cat and removed the responsibility of the Portion from both the Cat and Dog classes, making them more Single Responsibility Principle compliant.
