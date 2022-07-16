---
layout: post
title:  "GoLang: Decouple Depth First Tree Traversal from Business Logic"
date:   2022-07-16 11:00:00
categories: golang
uid: dd4
ads: false
comments: true
tags:
- "go"
- "golang"
- "programming"
- "decoupling"
---

If you have been in this industry for any amount of
time, you probably have heard about **decoupling**. 

**What is decoupling?** *Decoupling in programming is making parts of your code independent (or at least less dependent!) on other parts of your code.*

**Why is decoupling good?** Let me show you.

Consider the following Go function where we traverse a tree. Note that the `type` definitions for `Node` and `Stack` are not shown. If you wish to see them, they will be at the end of this post.*

In this example, all we do is traverse every node in the Tree and print its value. 

{% highlight go %}

func depthFirstTreeTraversalCoupled(root *Node) {
  stack := &Stack{}
  stack.Push(root)

  for !stack.IsEmpty(){
    node := stack.Pop()

    // do fancy business logic with the node
    // or just print the value
    fmt.Println(node.Value)
    
    for _, child := range node.Children {
      stack.Push(child)
    }
  }
}

{% endhighlight %}

We could replace the `fmt.Println(node.Value)` with fancier business logic like summing the node values. Let's do that.

{% highlight go %}

func depthFirstTreeTraversalCoupled(root *Node) {
  stack := &Stack{}
  stack.Push(root)

  sum := 0

  for !stack.IsEmpty(){
    node := stack.Pop()

    // do fancy business logic with the node
    // or just print the value
    sum = sum + node.Value
    
    for _, child := range node.Children {
      stack.Push(child)
    }
  }

  fmt.Println(sum)
}

{% endhighlight %}

Unfortunately, we had to initialize the `sum` variable. Additionally, we had to "do something" with sum at the end so I added `fmt.Println(sum)` at the end. Alternatively, we could return the `sum` which would change the function to be as follows:

{% highlight go %}

func depthFirstTreeTraversalCoupled(root *Node) int {
  stack := &Stack{}
  stack.Push(root)

  sum := 0

  for !stack.IsEmpty(){
    node := stack.Pop()

    // do fancy business logic with the node
    // or just print the value
    sum = sum + node.Value
    
    for _, child := range node.Children {
      stack.Push(child)
    }
  }

  fmt.Println(sum)
  return sum
}

{% endhighlight %}

Great, so we've returned the `sum`! Actually, not so great. For one, the name `depthFirstTreeTraversalCoupled` no longer makes sense. It would be better named `sumTreeValues`. We can rename it to that, but what if our codebase still needs the original function? We would need to have two functions with very similar algorithms doing very different things.

So let's fix this by decoupling the depth first tree traversal algorithm from the business logic. In this case, the business logic is either printing the values or summing the values.

What we will do is instead of baking the business logic directly into the function, we will pass it in as a function parameter. 

{% highlight go %}

func depthFirstTreeTraversal(root *Node,  businessLogic func(*Node)) {
  stack := &Stack{}
  stack.Push(root)

  for !stack.IsEmpty(){
    node := stack.Pop()
    businessLogic(node)
    for _, child := range node.Children {
      stack.Push(child)
    }
  }
}

{% endhighlight %}

If we want to simply print everything, we do that as follows:

{% highlight go %}

func main() {
  depthFirstTreeTraversal(tree, func(n *Node) {
    fmt.Println(n.Value)
  })
}

{% endhighlight %}

Alternatively, we could create a `printAllTreeValues` function like this:

{% highlight go %}

func printAllTreeValues(tree *Node) {
  depthFirstTreeTraversal(tree, func(n *Node) {
    fmt.Println(n.Value)
  })
}

{% endhighlight %}

If we want to do a sum, we do it as follows:

{% highlight go %}

func main() {
  sum := 0
  depthFirstTreeTraversal(tree, func(n *Node) {
    sum = sum + n.Value
  })

  fmt.Println(sum)
}

{% endhighlight %}

This also can be put into a function.

{% highlight go %}

func sumAllTreeValues(tree *Node) int {
  sum := 0
  depthFirstTreeTraversal(tree, func(n *Node) {
    sum = sum + n.Value
  })

  fmt.Println(sum)
  return sum
}

{% endhighlight %}

So why was all of this helpful? Because it allows us to write the `depthFirstTreeTraversal` only once. So that removes quite a few lines of code if your doing a bunch of tree traversals! Additionally, modifying the code is much easier because you only have to modify the business logic functions and not `depthFirstTreeTraversal`. 

**Can I use this during a coding interview**? Absolutely! If you get a a coding question that requires the use of a well-defined algorithm like depth first traversal, then decoupling your code like this should increase your score with the interviewer! Just make sure you tell them why you are doing it. Now, you can almost definitely solve your coding problem without doing this. You might run out of time before you actually are able to refactor your code to be more decoupled. You can still get some points potentially if you quickly explain to your interview how you would decouple the code.

**Is this code fully decoupled?** No. The algorithm in this case is still heavily constrained by the types `Node` and `Stack`. How would you decouple the algorithm from those types? Here are their definitions:

{% highlight go %}

type Node struct {
  Value int
  Children []*Node
}

type Stack struct {
  items []*Node
}

func (stack *Stack) Push(node *Node) {
  if (stack.items == nil) {
    stack.items = []*Node{}
  }

  stack.items = append(stack.items, node)
}

func (stack *Stack) Pop() *Node {
  if len(stack.items) == 0 {
    return nil
  }
  top := stack.items[len(stack.items)-1]
  stack.items = stack.items[0:len(stack.items)-1]
  return top
}

func (stack *Stack) IsEmpty() bool {
  return len(stack.items) == 0
}

{% endhighlight %}

I hope you enjoyed reading!