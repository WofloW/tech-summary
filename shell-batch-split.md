Bash shell commands 

Iterate files in folder 
Split the filename
Create folder
Execute script on the file and save the output to log file
```bash
#!/bin/bash

# filename format
# filename="bob_202001.zip"
for filename in /Downloads/*.zip; do
	IN=$filename

	IFS=\/ read -a fields <<<"$IN"

	IFS=/_ read -a names <<<"${fields[2]}"
	name=${names[0]}

	IFS=/. read -a dates <<<"${names[1]}"
	date=${dates[0]}
	# after this command, the IFS resets back to its previous value (here, the default):
	# IFS=$' \t\n'
	set | grep ^IFS=

	mkdir -p $date
  # > can save the result to file but tee can display and save the result to file at the same time
	sh ./<your-script>.sh $IN | tee ./$date/$name-$date.log
done
```
