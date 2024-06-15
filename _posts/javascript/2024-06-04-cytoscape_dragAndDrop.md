---
layout: post
title: "[JavaScript] cytoscape compound-drag-and-drop 구현"
date: 2024-06-04 13:00:23 +0900
categories: "JavaScript"
tag: ["JavaScript", "svg"]
---

![cytologo](https://github.com/bong0716/photogram/assets/119990564/d3c6d9d1-93ee-4270-b429-0f6cee4139fd)

<br>

cytoscape.js란 네트워크 데이터의 시각화 및 분석을 위한 JavaScript 라이브러리다. 원하는 레이아웃이 있어 사용했지만 구글에 자료가 별로 없다는 단점이 있다 ㅠㅠ

<br>

npm을 사용하기 때문에 node.js를 설치해야 한다. npm을 사용하고 싶지 않으면 필요한 플러그인 파일을 프로젝트에 포함시키면 된다. 나같은 경우 VScode에서 Live Server를 사용하는 게 편해 후자를 택했다.    

<br>

**구현해볼 기능은 노드를 다른 레이아웃으로 이동시키는 것이다. 즉 Drag&Drop을 구현하려 한다.**

<br>

--- 

플러그인은 [여기서](https://github.com/cytoscape/cytoscape.js-compound-drag-and-drop/blob/master/cytoscape-compound-drag-and-drop.js) cytoscape-compound-drag-and-drop.js 파일을 가져왔다. 


구현 코드는 [여기를](https://codesandbox.io/p/sandbox/cytoscape-demo-forked-eqb4nj?file=%2Findex.html%3A11%2C26) 참고했다. 

<br>

![js](https://github.com/bong0716/photogram/assets/119990564/403f93ac-68dd-46ca-9ce3-b02f374c1ff5)
<p align="center">(js 디렉토리 아래 플러그인 파일들을 위치시킨 후)</p> <br>

<p align="center"><img src="https://github.com/bong0716/photogram/assets/119990564/90879c47-7d31-45e5-b156-13b6d40219a9"></p>
<p align="center">(html 파일 header 안에 넣어줬다.)</p> <br>
이게 번거로우면 node.js를 설치하고 [여기 나온 것처럼](https://codesandbox.io/p/sandbox/cytoscape-demo-forked-eqb4nj?file=%2Findex.html%3A11%2C26) 자바스크립트 파일에 플러그인을 import하면 된다.

<br>

## 추가된 코드
---

**[1. cytoscape 객체에 compoundDragAndDrop 플러그인 추가]**
```javascript
try {
  cytoscape.use(compoundDragAndDrop);
} catch (e) {}
```
cytoscape 라이브러리에 compoundDragAndDrop 플러그인을 추가하고, 에러가 발생 시 무시하도록 해놨다. 

---

<br>

**[2. 그래프에 compoundDragAndDrop 플러그인 적용]**
```javascript
var cy = cytoscape({
  container: document.getElementById("cy"),
  ready: function () {
    this.compoundDragAndDrop();
  },
})
```

---

<br>

**[3. Drag&Drop 코드 추가]**
```javascript
var cdnd = cy.compoundDragAndDrop();
var removeEmptyParents = false;

var isParentOfOneChild = function (node) {
  debugger;
  return node.isParent() && node.children().length === 1;
};

var removeParent = function (parent) {
  debugger;
  parent.children().move({ parent: null });
  parent.remove();
};

var removeParentsOfOneChild = function () {
  debugger;
  cy.nodes().filter(isParentOfOneChild).forEach(removeParent);
};

// custom handler to remove parents with only 1 child on drop
cy.on("cdndout", function (event, dropTarget) {
  debugger;
  if (removeEmptyParents && isParentOfOneChild(dropTarget)) {
    removeParent(dropTarget);
  }
});

// custom handler to remove parents with only 1 child (on remove of drop target or drop sibling)
cy.on("remove", function (event) {
  debugger;
  if (removeEmptyParents) {
    removeParentsOfOneChild();
  }
});
```

이렇게 하면 각 노드에 Drag&Drop 기능이 생긴다. 그런데 문제는 빈 부모노드에 자식노드를 drop 했을 때, 부모노드와 자식노드를 자식으로 가지는 새로운 부모노드가 생성된다는 점이다. 이 부분을 수정하기 위해 다음 코드를 추가했다. cytoscape-compound-drag-and-drop.js를 봐야한다! 

---

<br>

**[cytoscape-compound-drag-and-drop.js 기존 코드]**
```javascript
if (overlappingNodes.length > 0) {
        // potential parent
        var overlappingParents = overlappingNodes.filter(isParent);

        var _parent, _sibling;

        if (overlappingParents.length > 0) {
          _sibling = cy.collection();
          _parent = overlappingParents[0]; // TODO maybe use a metric here to select which one
        } else {
          _sibling = overlappingNodes[0]; // TODO maybe use a metric here to select which one

          _parent = cy.add(options.newParentNode(_this.grabbedNode, _sibling));
        }

        _parent.addClass('cdnd-drop-target');

        if (_parent !== _sibling) {
          _parent.addClass('cdnd-new-parent');
        }

        _sibling.addClass('cdnd-drop-sibling');

        setParent(_sibling, _parent);
        _this.dropTargetBounds = getBoundsCopy(_parent, options.boundingBoxOptions);
        setParent(_this.grabbedNode, _parent);
        _this.dropTarget = _parent;
        _this.dropSibling = _sibling;

        _this.grabbedNode.emit('cdndover', [_parent, _sibling]);
      }
```
294번째 줄부터 보면 된다. 

- overlappingNodes : 특정 노드를 다른 노드 위로 드롭할 때, 그 노드와 겹치는 모든 노드를 담고 있는 컬렉션 
- _sibling : 형제노드

- addClass('cdnd-drop-target') : 드롭 대상 부모 노드임을 나타냄.
- addClass('cdnd-new-parent') : 새로운 부모 노드임을 나타냄. 기존 부모노드가 아닌 새로운 부모 노드라는 뜻.

- addClass('cdnd-drop-sibling) : 드롭된 노드의 형제임을 나타냄.

- setParent(_sibling, _parent) : _sibling의 부모를 _parent로 설정.
부모-자식 관계를 설정하는 데 사용됨. 

- setParent(_this.grabbedNode, _parent) : 드롭된 노드의 부모를 _parent로 설정

<br>

**[cytoscape-compound-drag-and-drop.js 변경 코드]**
```javascript
    if (overlappingParents.length > 0) {
          _sibling = cy.collection();
          _parent = overlappingParents[0]; // TODO maybe use a metric here to select which one
        } else {
          _sibling = overlappingNodes[0]; // TODO maybe use a metric here to select which one
        
          // 클러스터 노드인지 확인
          if (_sibling.hasClass('cluster')) {  // 추가된 코드 
            _parent = _sibling;
          } else {
            // 클러스터 노드가 아니면 새로운 부모 노드 생성
            _parent = cy.add(options.newParentNode(_this.grabbedNode, _sibling));
          }
        }
```

---

![수정 전](https://github.com/bong0716/photogram/assets/119990564/c95557a6-0d0b-4cd1-8460-b80799134a97)
<p align="center">(수정 전)</p> <br>

![수정 전](https://github.com/bong0716/photogram/assets/119990564/b14a0691-d760-4a30-9aa0-5cc50c55a89c)
<p align="center">(수정 후)</p> <br>

---

xml_itms 레이아웃에 Approve.xml 자식노드가 잘 drop 되는 것을 확인!