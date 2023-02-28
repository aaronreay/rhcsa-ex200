# 1. Conditionally execute code ( if, test, [], etc. )
We preface our bash scripts with a shebang, in this case `#!/bin/bash`
```
# exit code of 0, success. non-zero is considered a failure
#!/bin/bash

echo 'Enter score'
read x

if [[ $x == 70 ]]; then
  echo 'success'
  exit 0
else
  exit 1
fi
```
Above we have a simple if statement. If the value of `x` is 70, as determined by the `read` function, we will exit
with an exit code of 0, else we will exit with 1, and fail.

# 2. Use looping constructs (for, etc.) to process file, command line output
* `while loop` - continue until the condition is no longer set
```
#!/bin/bash
counter=1
while [ $counter -le 10 ]
do
  echo $counter
  (( counter++ ))
done
```
Above, we will start at one, and continue looping until the condition of 9 is met, as this is less than 10

* `for loop` - will run a set number of times, as defined by the user
```
#!/bin/bash
for num in {1..20}
do
  echo $num
done
```
Above we will loop 20 times, as defined by out `for num in {1..20}` statement

# 3. Process script inputs ($1, $2, etc.)
* `$0` - name of the script
* `$1..9` - 1st through to the 9th variables
We can also do `$10` onwards, but this will need to be in-closed, `${10}`

# 4. Processing output of shell commands within a script
To process shell commands within a script, we must first assign these to a useable variable
`date=`date +%Y-%m-%d` - this will assign a variable with the value of the `date` function, using it's special paramters of `%Y%m%d`. Look at `man date` for more info
```
#!/bin/bash
today=`date`
echo "Today is $today"
```

