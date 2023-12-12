```shell
Fern=/Users/anner/Fern/
Moss=/Users/anner/Moss/
directories=("AIGC" "书籍" "Fun" "Web3" "算法" "架构" "折腾" "开发" "思维")

for directory in "${directories[@]}"; do
    cp -r "${Moss}${directory}" "${Fern}content/"
done

cd /Users/anner/Fern && npx quartz sync && cd -
```