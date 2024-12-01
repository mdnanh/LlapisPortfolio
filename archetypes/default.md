+++
date = '{{ .Date.Format "Jan, 2006" }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
+++
