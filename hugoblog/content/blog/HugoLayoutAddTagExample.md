+++
showonlyimage = false
draft = false
date = "2023-05-29T29:25:22+05:30"
title = "Hugo自訂Tag Filter功能筆記"
weight = 1
showmoretile = false
tags = "Hugo,Web"
+++


[Hugo文件](https://gohugo.io/documentation/)  

#### 1. 打開主題當中/layout 目標html
![結果示意][1]

#### Hugo html相關語法:
##### 1. 宣告變數:
[文件](https://gohugo.io/templates/introduction/#example-2-declaring-a-variable-name-for-an-array-elements-value)  
{{ $變數 := "某值" }}
```html
<!-- 宣告$myVar變數 = .Params.tags.split(',') -->
{{ $myVar := split .Params.tags "," }}
```  
* * *  
  
##### 2. Range陣列資料陣列資料案染:  
[文件](https://gohugo.io/functions/range/)  
後面需要{{end}}作結尾包覆
```html
<!-- 取得index, element寫法 -->
{{ range $index, $element := $myVar }}
    <span>這是Index{{$index}}</span>
    <span>這是Element{{$element}}</span>
{{ end }}

<!-- 快速寫法 -->
{{ range $myVar }}
    <span>這是Element{{ . }}</span>
{{ end }}
```  
* * *  

##### 3. if else判斷式 + eq condition判斷:
[if文件](https://gohugo.io/templates/introduction/#example-3-if)  
[eq文件](https://gohugo.io/functions/eq/)  
{{if}}最後需要{{end}}作結尾包覆  
eq 相當於JS中 ==, 只是2比對值放在它之後,中間用空格區分  
($myVar | len) == $myVar.length
```html
<!-- 取得index, element寫法 -->
{{ if eq ($myVar | len) 0 }}
    <span>這是0</span>   
{{ if eq ($myVar | len) 1 }}
    <span>這是1</span>   
{{ else }}
    <span>這是Others</span>
{{ end }}
```  
* * *  

##### 4. len長度: array/dict/string:  
[文件](https://gohugo.io/functions/len/)  
{{if}}最後需要{{end}}作結尾包覆  
eq 相當於JS中 ==, 只是2比對值放在它之後,中間用空格區分  
($myVar | len) == $myVar.length
```html
<!-- String 2 -->
{{ "ab" | len }}

<!-- String 0 -->
{{ "" | len }}

<!-- Array 2 -->
{{ slice "a" "b" | len }}

<!-- Array 0 -->
{{ slice | len }} → 0

<!-- Dict/Obj 2 -->
{{ dict "a" 1 "b" 2  | len }}

<!-- Dict/Obj 0 -->
{{ dict | len }} → 0
```  

* * *  
#### 完整html demo:  
```html
<div class="col-xs-12 col-sm-8 col-md-9 content-column">
    <div class="clearButtons"></div>
    {{ partial "mobile_nav_toggle.html" . }}
    <div class="grid">
        <div class="row">
          {{ range .Data.Pages }}
              <div class="col-xs-12 col-sm-6 col-md-4 col-lg-3 masonry-item" tagData="{{.Params.tags}}">
                  <div class="box-masonry">
                      {{ if and (isset .Params "image") .Params.image }}

                        {{ if (isset .Params "usedirecturl") }} <!-- 直接倒到外部WebSite -->
                        <a target="_blank" href="{{ .Params.useDirectUrl }}" title="" class="box-masonry-image with-hover-overlay">
                        {{ else if eq .Params.showonlyimage true }}
                        <a href="{{ .Permalink }}" title="" class="box-masonry-image with-hover-overlay">
                        {{ else }}
                        <a href="{{ .Permalink }}" title="" class="box-masonry-image with-hover-overlay with-hover-icon">
                        {{ end }}
                            <img src="{{.Params.image | absURL}}" alt="" class="img-responsive">
                        </a>
                      {{ end }}
                      {{ if eq .Params.showonlyimage true }}
                      <div class="box-masonry-hover-text-header">
                      {{ else }}
                      <div class="box-masonry-text">
                      {{ end }}
                          <h4>
                            {{ if (isset .Params "usedirecturl") }} <!-- 直接倒到外部WebSite -->
                            <a target="_blank" href="{{ .Params.useDirectUrl }}">
                            {{ else }}
                            <a href="{{ .Permalink }}">
                            {{ end }}
                            {{ .Title }}
                            </a>
                          </h4>

                          {{ if (isset .Params "tags") }} <!-- 做tags Filter功能 -->
                            {{ $myVar := split .Params.tags "," }}
                            {{ range $index, $element := $myVar }}
                            
                            {{ if eq (mod $index 4) 0 }}
                            <button type="button" class="chooseTag btn-sm btn btn-info" style="margin-top: 8px ;" tag="{{ $element }}">
                            {{ else if eq (mod $index 4) 1 }}
                            <button type="button" class="chooseTag btn-sm btn btn-primary" style="margin-top: 8px ;" tag="{{ $element }}">
                            {{ else if eq (mod $index 4) 2 }}
                            <button type="button" class="chooseTag btn-sm btn btn-warning" style="margin-top: 8px ;" tag="{{ $element }}">
                            {{ else }}
                            <button type="button" class="chooseTag btn-sm btn btn-secondary" style="margin-top: 8px ;" tag="{{ $element }}">
                            {{end}}
                              {{$element}}
                            </button>
                            {{end}}
                          {{end}}

                          {{ if and (isset .Params "showmoretile") .Params.showmoretile }}
                            <!-- 把 Description 關掉 -->
                            <div class="box-masonry-description">
                              <p>
                                  {{ if .Description }}
                                    {{ .Description }}
                                  {{ else }}
                                    {{ .Summary }}
                                  {{ end }}
                              </p>
                            </div>
                          {{end}}
                      </div>
                  </div>
              </div>
          {{ end }}
        </div>
    </div>
</div>
```

* * *  
#### 完整JS demo:  
```js
<script>
  let selectedTag = null; //- 以選擇Tags(Array string)
  const btnsFilter = document.querySelectorAll('button.chooseTag'); //- Tag Button ele
  const allPanel = document.querySelectorAll('div.masonry-item'); //- Render Context
  const clearBtnDiv = document.getElementsByClassName('clearButtons')[0]; //- clear Tag button area

  btnsFilter.forEach(function(button) { //- Filter加入Tags
    button.addEventListener('click', function() {
      var theTag = button.getAttribute('tag');
      if(selectedTag==null)selectedTag=[theTag]; //- isFirst element
      else if(selectedTag.includes(theTag)){ //- 已加入
        return;
      }
      else{ //- 加入
        selectedTag.push(theTag);
      }
      AddClearButtonRender(theTag);
      filterRender();
    });
  });

  function AddClearButtonRender(tagName){ //- 加上display:none, display:block
    var button = document.createElement('button');
    button.textContent = tagName;

    button.addEventListener('click', function() { //- 觸發移除自己Tag
      button.parentNode.removeChild(button); // remove <Button> ele
      if(selectedTag==null)return;
      var idx = selectedTag.indexOf(tagName);
      if(idx>=0){
        selectedTag.splice(idx,idx+1);
        if(selectedTag.length == 0)selectedTag = null; //- if no conditions
        filterRender(); //- ReRender context
      }
    });
    
    button.classList.add('btn','btn-primary'); //- add button css
    button.setAttribute('type', "button");
    button.setAttribute('tag', tagName);
    clearBtnDiv.appendChild(button);
  }

  function filterRender(){ //- 加上display:none, display:block
    const choosedTag = selectedTag==null ? null : selectedTag;
    allPanel.forEach(function(ele) {
      var eleHasTags = ele.getAttribute('tagData');
      var isShow = true;
      if(selectedTag){
        selectedTag.forEach(tg=>{
          if(!eleHasTags.includes(tg)){
            isShow=false;
            return;
          }
        });
      }
      ele.style.display = isShow ? "block":"none";
    });
  }

</script>
```  

[1]: /img/blog/HugoPortofolio.JPG
