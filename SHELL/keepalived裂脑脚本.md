```

#!/bin/bash
while true
ping -c 2 -W 3 192.168.21.69 &>/dev/null
if [ $? -eq 0 -a `ip add | grep 192.168.21.69 | wc -l` -eq 1 ]
then
    echo "ha is split brain warning"
else
    echo "ha is OK"
fi
sleep 5
done
