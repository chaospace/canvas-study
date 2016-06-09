## Canvas 마우스 hitTest 내용정리

 **1. 대상 area와 마우스 좌표를 직접 비교(고전적)**
 ```{.javascript}
 context.rect( myRect.x , myRect.y, myRect.w, myRect.h );//사각형을 그림
 var area = screen.getBoundingClientRect();
 //마우스 좌표 정
 var x = e.clientX - area.left;
 var y = e.clientY - area.top;  
 var hitted = ( x >= myRect.x && x <= myRect.x + myRect.width &&
                y >= myRect.y && y <= myRect.y + myRect.height)
 ```

 **2. 제공API를 이용하는 법(isPointInPath)**
 ```{.javascript}
 // isPointInPath를 이용해 포인트가 path에 포함되는지 체크
 if(context.isPointInPath( x, y )){
   context.fillStyle = "red";
 }
 context.fill();
 ```
간단하게 이용하기는 좋지만 체크해야 되는
아이템이 많아지면 더미패스를 그리기 위한 부하가 좀 아쉬움.

 **3. context의 getImageData를 이용하는 법**
  getImageData를 통해 가져온 픽셀정보에서
  해당좌표의 픽셀 alpha값이 0이 아니면 히트라고 간주
  ```{.javascript}
  imageData = ctx.getImageData( x, y, 1, 1 );
  results = (imageData.data[3] > 0) ? true : false;
  ```

## Canvas를 사용하며 느낀점
다양한 드로잉 오브젝트를 다루기 위해서는 객체 관리가 필수.
객체에서는 내부 canvas를 가지고 original정보를 가지고 있는게 퍼포먼스에 좋음.
```{.javascript}
// 위치 변경전의 오리지널 draw
protected _drawOriginItem():void{
  this._clearOrigin();
  this.context.save();
  this.context.beginPath();
  this.context.fillStyle = this.vo.color;
  this.context.arc( this.vo.radius,
                    this.vo.radius,
                    this.vo.radius,
                    0,
                    Math.PI*2);
  this.context.fill();
  this.context.closePath();
  this.context.restore();
}

protected _copyOfOrigin( ctx:CanvasRenderingContext2D ):void {
    ctx.drawImage(this.canvas,
                  this.vo.x-this.vo.radius,
                  this.vo.y-this.vo.radius, this.vo.radius*2, this.vo.radius*2);
}

// 랜더링 시는 original을 옮겨옴.
render( ctx:CanvasRenderingContext2D ):void{
  this._copyOfOrigin( ctx );
}
```
다양한 표현을 위한 layer계층을 만들어 두는게 좋다.
~~~
  - 배경2 레이어
  - 배경1 레이어
  - 객체 레이어
~~~
