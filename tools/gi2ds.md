# gi2ds - Google Image Search 2 Dataset
**gi2ds** 는 구글 이미지 검색에 기반하여 이미지 데이터셋을 생성할 때 도움을 줄 수 있도록 디자인되었다. 이미지를 클릭함으로써 연관성이 적은 이미지를 포함하거나 제외할 수 있게 해 준다. 이미지의 URL 목록은 우측 하단에 표시가 된다. 더 많은 이미지를 얻고 싶다면 화면을 아래로 스크롤링해서, 더 많은 이미지를 불러오거나 `show more results` 버튼을 클릭한 후 계속해서 스크롤을 내리면 된다.

![gi2ds - Google Image Search to Dataset](https://github.com/toffebjorkskog/ml-tools/blob/master/images/gi2ds-usage.png?raw=true)

## 사용을 시작해 보는법
[아래쪽에 설명된 북마크릿(bookmarkelt)](#bookmarklet) 을 사용하거나, 이미지 검색이 이뤄진 페이지에서, 브라우져의 자바스크립트 콘솔을 실행한 후 아래의 자바스크립트 코드를 붙여넣는 두 가지 방법이 있다.

```javascript
(function(e, s) {
    e.src = s;
    e.onload = function() {
        jQuery.noConflict();
        jQuery('<style type="text/css"> .remove { opacity:0.3;}\n .urlmodal {padding: 10px; background-color: #eee; position: fixed; bottom: 0; right: 0; height: 100px; width: 300px; z-index: 1000;} .urlmodal textarea {width: 100%; height: 250px;}</style>').appendTo('head');
        jQuery('<div class="urlmodal"><h3>Let\'s create a dataset</h3><textarea>Scoll all the way down\nClick "Show more images"\nScroll more\nClick on the images you want to remove from the dataset\nThe urls will appear in this box for you to copy.</textarea></div>').appendTo('body');

        jQuery('#rg').on('click', '.rg_di', function() {
            jQuery(this).toggleClass('remove');
            updateUrls();
            return false;
        });
        jQuery(window).scroll(updateUrls);
        jQuery('.urlmodal textarea').focus(function() {updateUrls(); setTimeout(selectText, 100)}).mouseup(function() {return false;});

        function updateUrls() {
            var urls = Array.from(document.querySelectorAll('.rg_di:not(.remove) .rg_meta')).map(el=>JSON.parse(el.textContent).ou);
            var search_term = jQuery('.gsfi').val();
            jQuery('.urlmodal textarea').val(urls.join("\n"));
            jQuery('.urlmodal h3').html(search_term + ": " + urls.length);
        }

        function selectText() {
            jQuery('.urlmodal textarea').select();
        }
    };
    document.head.appendChild(e);
})(document.createElement('script'), '//code.jquery.com/jquery-latest.min.js');
```

## 북마크로 추가하는 방법
<a name="bookmarklet"></a>
아래의 코드 스니펫을 [북마크 추가하기](images/gi2ds-add-bookmark.png) 에 설명된 방식대로, 추가하여 북마크를 생성한다.

```javascript:(function()%7B(function(e%2C%20s)%20%7Be.src%20%3D%20s%3Be.onload%20%3D%20function()%20%7BjQuery.noConflict()%3BjQuery('%3Cstyle%20type%3D%22text%2Fcss%22%3E%20.remove%20%7B%20opacity%3A0.3%3B%7D%5Cn%20.urlmodal%20%7Bpadding%3A%2010px%3B%20background-color%3A%20%23eee%3B%20position%3A%20fixed%3B%20bottom%3A%200%3B%20right%3A%200%3B%20height%3A%20100px%3B%20width%3A%20300px%3B%20z-index%3A%201000%3B%7D%20.urlmodal%20textarea%20%7Bwidth%3A%20100%25%3B%20height%3A%20250px%3B%7D%3C%2Fstyle%3E').appendTo('head')%3BjQuery('%3Cdiv%20class%3D%22urlmodal%22%3E%3Ch3%3ELet%5C's%20create%20a%20dataset%3C%2Fh3%3E%3Ctextarea%3EScoll%20all%20the%20way%20down%5CnClick%20%22Show%20more%20images%22%5CnScroll%20more%5CnClick%20on%20the%20images%20you%20want%20to%20remove%20from%20the%20dataset%5CnThe%20urls%20will%20appear%20in%20this%20box%20for%20you%20to%20copy.%3C%2Ftextarea%3E%3C%2Fdiv%3E').appendTo('body')%3BjQuery('%23rg').on('click'%2C%20'.rg_di'%2C%20function()%20%7BjQuery(this).toggleClass('remove')%3BupdateUrls()%3Breturn%20false%3B%7D)%3BjQuery(window).scroll(updateUrls)%3BjQuery('.urlmodal%20textarea').focus(function()%20%7BupdateUrls()%3B%20setTimeout(selectText%2C%20100)%7D).mouseup(function()%20%7Breturn%20false%3B%7D)%3Bfunction%20updateUrls()%20%7Bvar%20urls%20%3D%20Array.from(document.querySelectorAll('.rg_di%3Anot(.remove)%20.rg_meta')).map(el%3D%3EJSON.parse(el.textContent).ou)%3Bvar%20search_term%20%3D%20jQuery('.gsfi').val()%3BjQuery('.urlmodal%20textarea').val(urls.join(%22%5Cn%22))%3BjQuery('.urlmodal%20h3').html(search_term%20%2B%20%22%3A%20%22%20%2B%20urls.length)%3B%7Dfunction%20selectText()%20%7BjQuery('.urlmodal%20textarea').select()%3B%7D%7D%3Bdocument.head.appendChild(e)%3B%7D)(document.createElement('script')%2C%20'%2F%2Fcode.jquery.com%2Fjquery-latest.min.js')%7D)()```

새로운 북마크의 이름을 gi2ds 라고 지정할 수도 있다 (원하는대로 정하면 된다).

![북마크 추가하기](https://github.com/toffebjorkskog/ml-tools/blob/master/images/gi2ds-add-bookmark.png?raw=true)

![코드 스니펫 붙여넣기](https://github.com/toffebjorkskog/ml-tools/blob/master/images/gi2ds-paste-snippet.png?raw=true)

북마크를 추가하면, 자바스크립트 콘솔에 직접적으로 코드를 붙여넣는것 대신에 누를 수 있는 버튼이 될 것이다.

![추가된 북마크](https://github.com/toffebjorkskog/ml-tools/blob/master/images/gi2ds-bookmarklet-button.png?raw=true)

### 이 툴을 만들게된 계기
현재 내가 인터네셔널 펠로우로 참석하고 있는, 올해의 [fast.ai](https://www.fast.ai/) 코스 (v3) 로부터 영감을 받았다. 이 코스는 2019년 1월에 대중에게 공개될 예정이다.
