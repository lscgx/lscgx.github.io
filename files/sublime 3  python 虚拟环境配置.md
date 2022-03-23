##### sublime 3  python 虚拟环境配置

1. 新建一个bulid

   <img src="https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL2ltZzIwMjAuY25ibG9ncy5jb20vYmxvZy8xOTgwMTA4LzIwMjAwMy8xOTgwMTA4LTIwMjAwMzI0MDI1NDA5OTMzLTUwMTQ2Nzk4OS5wbmc=.jpg" width="80%" />

   2. 新建文件，输入文件名（例：python(pytenv)）并保存

      <img src="https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL2ltZzIwMjAuY25ibG9ncy5jb20vYmxvZy8xOTgwMTA4LzIwMjAwMy8xOTgwMTA4LTIwMjAwMzI0MDI1NzQwMzU2LTE3ODEzMzQ5MjUucG5n.jpg" width="80%" />

      ```shell
      {
          "shell":true,
          "cmd": ["C:\\Users\\dy040\\pytenv\\Scripts\\python.exe", "$file"],
          "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
          "selector": "source.python",
          "variants":
      	[
      		{
      			"name": "Run",
      			"cmd": ["start", "cmd", "/c", "C:\\Users\\dy040\\pytenv\\Scripts\\python.exe $file &pause"]
      		}
      	]
      }
      ```
      
      3. 运行时切换环境即可
      
      

