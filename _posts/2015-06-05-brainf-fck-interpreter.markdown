---
layout: post
title:  "Writing my first Interpreter"
date:   2015-06-05 18:25:00
categories: recurse-center
comments_enabled: true
---

Talking with [Brad][1] on Wednesday, he showed to me an [Interpreter for BrainFuck][2] he written in Go. I really liked his idea, so I also implemented mine but in JS (I have never written or seen before an interpreter).

# The interpreter #

I am impresed which such a cool name there is for a switch statement with some bunch of extra variables. 

Since Brain F* only contains few keywords: '>', '<', ']', '[', '+', '-', ',' and '.'. It was very quick to implement it.

Now I am wondering if all the *interpreted languages* just have a huge switch for each keyword? I will have to figure this out!

## My interpreter code ##

```javascript
	this.run = function(){
		while(instruction_pointer<instructions.length){
			switch(instructions[instruction_pointer]) {
				case '>':
					data_pointer = data_pointer + 1;
					break;
				case '<':
					data_pointer = data_pointer - 1;
					break;
				case '+':
					memory[data_pointer] = memory[data_pointer] + 1;
					break;
				case '-':
					memory[data_pointer] = memory[data_pointer] - 1;
					break;
				case '.':
					output = String.fromCharCode(memory[data_pointer]);
					var current_output = document.getElementById("output").innerHTML;
					current_output = current_output + output;
					document.getElementById("output").innerHTML = current_output;
					console.log(String.fromCharCode(memory[data_pointer]));
					break;
				case ',':
					var character = prompt("Input a character to be stored");
					memory[data_pointer] = character.charCodeAt(0);
					break;
				case '[':
					if (memory[data_pointer] == 0) {
						instruction_pointer = this.findMatching(instruction_pointer);
						continue;
					}
					break;
				case ']':
					if (memory[data_pointer] != 0){
						instruction_pointer = this.findMatching(instruction_pointer);
						continue;
					}
					break;
			}
			instruction_pointer = instruction_pointer + 1;
		}
	}
```

#Live run your Brain F*ck code below #

## Brain F*ck code goes here! ##

<script src="public/interpreter.js"></script>

<form>
	<textarea id="input" rows="20" cols="72">++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.</textarea>
	</br>
	<button type="button" onclick="process_file()">Run!</button>
</form>

## Output of the program ##

<p id="output"></p>

### Thanks a lot Brad for the idea and help during the implementation! ###

[1]: https://twitter.com/daveypocket
[2]: https://github.com/DaveyPocket/BFinterpreter