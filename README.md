# Queue





A queue is a list where you can only insert new items at the back and remove items from the front. This ensures that the first item you enqueue is also the first item you dequeue. First come, first serve!
Why would you need this? Well, in many algorithms you want to add objects to a temporary list and pull them off this list later. Often the order in which you add and remove these objects matters.
A queue gives you a FIFO or first-in, first-out order. The element you inserted first is the first one to come out. It is only fair!

the first element is inserted from one end called the REAR(also called tail), and the removal of existing element takes place from the other end called as FRONT(also called head).
Which is exactly how queue system works in real world. If you go to a ticket counter to buy movie tickets, and are first in the queue, then you will be the first one to get the tickets. Right? Same is the case with Queue data structure. Data inserted first, will leave the queue first.
The process to add an element into queue is called Enqueue and the process of removal of an element from queue is called Dequeue.
peek( ) function is oftenly used to return the value of first element without dequeuing it.

Applications of Queue
Queue, as the name suggests is used whenever we need to manage any group of objects in an order in which the first one coming in, also gets out first while the others wait for their turn, like in the following scenarios:
Serving requests on a single shared resource, like a printer, CPU task scheduling etc.
In real life scenario, Call Center phone systems uses Queues to hold people calling them in an order, until a service representative is free.
Handling of interrupts in real-time systems. The interrupts are handled in the same order as they arrive i.e First come first served.

Implementation of Queue Data Structure
Queue can be implemented using an Array, Stack or Linked List. The easiest way of implementing a queue is by using an Array.
Initially the head(FRONT) and the tail(REAR) of the queue points at the first index of the array (starting the index of array from 0). As we add elements to the queue, the tail keeps on moving ahead, always pointing to the position where the next element will be inserted, while the head remains at the first index.
implementation of queue
When we remove an element from Queue, we can follow two possible approaches (mentioned [A] and [B] in above diagram). In [A] approach, we remove the element at head position, and then one by one shift all the other elements in forward position.
In approach [B] we remove the element from head position and then move head to the next position.
In approach [A] there is an overhead of shifting the elements one position forward every time we remove the first element.
In approach [B] there is no such overhead, but whenever we move head one position ahead, after removal of first element, the size on Queue is reduced by one space each time.
Algorithm for ENQUEUE operation
Check if the queue is full or not.
If the queue is full, then print overflow error and exit the program.
If the queue is not full, then increment the tail and add the element.
Algorithm for DEQUEUE operation
Check if the queue is empty or not.
If the queue is empty, then print underflow error and exit the program.
If the queue is not empty, then print the element at the head and increment the head.


Here is a simplistic implementation of a queue in Swift. It is a wrapper around an array to enqueue, dequeue, and peek at the front-most item:
public struct Queue<T> {
  fileprivate var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }
  
  public var count: Int {
    return array.count
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }
  
  public mutating func dequeue() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeFirst()
    }
  }
  
  public var front: T? {
    return array.first
  }
}
This queue works well, but it is not optimal.

Complexity Analysis of Queue Operations
Just like Stack, in case of a Queue too, we know exactly, on which position new element will be added and from where an element will be removed, hence both these operations requires a single step.
Enqueue: O(1)
Dequeue: O(1)
Size: O(1)


Updates to optimize Queue
Enqueuing is an O(1) operation because adding to the end of an array always takes the same amount of time regardless of the size of the array.
You might be wondering why appending items to an array is O(1) or a constant-time operation. That is because an array in Swift always has some empty space at the end. If we do the following:
var queue = Queue<String>()
queue.enqueue("Ada")
queue.enqueue("Steve")
queue.enqueue("Tim")
then the array might actually look like this:
[ "Ada", "Steve", "Tim", xxx, xxx, xxx ]
where xxx is memory that is reserved but not filled in yet. Adding a new element to the array overwrites the next unused spot:
[ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
This results by copying memory from one place to another which is a constant-time operation.
There are only a limited number of unused spots at the end of the array. When the last xxx gets used, and you want to add another item, the array needs to resize to make more room.
Resizing includes allocating new memory and copying all the existing data over to the new array. This is an O(n) process which is relatively slow. Since it happens occasionally, the time for appending a new element to the end of the array is still O(1) on average or O(1) "amortized".
The story for dequeueing is different. To dequeue, we remove the element from the beginning of the array. This is always an O(n) operation because it requires all remaining array elements to be shifted in memory.
In our example, dequeuing the first element "Ada" copies "Steve" in the place of "Ada", "Tim" in the place of "Steve", and "Grace" in the place of "Tim":
before   [ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
                   /       /      /
                  /       /      /
                 /       /      /
                /       /      /
 after   [ "Steve", "Tim", "Grace", xxx, xxx, xxx ]
Moving all these elements in memory is always an O(n) operation. So with our simple implementation of a queue, enqueuing is efficient, but dequeueing leaves something to be desired...
A more efficient queue
To make dequeuing efficient, we can also reserve some extra free space but this time at the front of the array. We must write this code ourselves because the built-in Swift array does not support it.
The main idea is whenever we dequeue an item, we do not shift the contents of the array to the front (slow) but mark the item's position in the array as empty (fast). After dequeuing "Ada", the array is:
[ xxx, "Steve", "Tim", "Grace", xxx, xxx ]
After dequeuing "Steve", the array is:
[ xxx, xxx, "Tim", "Grace", xxx, xxx ]
Because these empty spots at the front never get reused, you can periodically trim the array by moving the remaining elements to the front:
[ "Tim", "Grace", xxx, xxx, xxx, xxx ]
This trimming procedure involves shifting memory which is an O(n) operation. Because this only happens once in a while, dequeuing is O(1) on average.

Here is how you can implement this version of Queue:
public struct Queue<T> {
  fileprivate var array = [T?]()
  fileprivate var head = 0
  
  public var isEmpty: Bool {
    return count == 0
  }

  public var count: Int {
    return array.count - head
  }
  
  public mutating func enqueue(_ element: T) {
    array.append(element)
  }
  
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }
    
    return element
  }
  
  public var front: T? {
    if isEmpty {
      return nil
    } else {
      return array[head]
    }
  }
}
The array now stores objects of type T? instead of just T because we need to mark array elements as being empty. The head variable is the index in the array of the front-most object.
Most of the new functionality sits in dequeue(). When we dequeue an item, we first set array[head] to nil to remove the object from the array. Then, we increment head because the next item has become the front one.
We go from this:
[ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
  head
to this:
[ xxx, "Steve", "Tim", "Grace", xxx, xxx ]
        head
It is like if in a supermarket the people in the checkout lane do not shuffle forward towards the cash register, but the cash register moves up the queue.
If we never remove those empty spots at the front then the array will keep growing as we enqueue and dequeue elements. To periodically trim down the array, we do the following:
    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }
This calculates the percentage of empty spots at the beginning as a ratio of the total array size. If more than 25% of the array is unused, we chop off that wasted space. However, if the array is small we do not resize it all the time, so there must be at least 50 elements in the array before we try to trim it.
source code  
https://github.com/Mo7yeeNotes/Queue
