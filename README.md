# imgUploadHtml5
#This is a use of HTML5 to achieve a plug-in, the purpose is to upload, reduce server pressure
<html><head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
<title>html5Upload</title>
<script type="text/javascript" src="http://code.jquery.com/jquery-compat-git.js"></script>
<style type="text/css">
    .container{
        width: 300px;
    }
    .resizer{
        overflow: hidden;
    }
    .resizer.have-img button.ok{
        display: inline-block;
    }
    .resizer.have-img .inner {
        display: block;
    }
    .inner{
        width: 100%;
        position: relative;
        font-size: 0;
        overflow: hidden;
        display: none;
    }
    img{
        width: 100%;
    }

    .frames{
        position: absolute;
        top: 0;
        left: 0;
        border: 1px solid black;
        cursor: move;
        outline: rgba(0, 0, 0, 0.6) solid 10000px;
    }
    button.ok{
        float:right;
        margin-left: 5px;
        display: none;
    }
    canvas{
        max-width: 100%;
        margin:auto;
        display: block;
    }

</style>
<script type="text/javascript">//<![CDATA[
$(window).on('load', function() {
    var tmp=$('<div class="resizer">'+
    '<div class="inner">'+
    '<img>'+
    '<div class="frames"></div>'+
    '</div>'+
        //'<button>&#10007;</button>'+
    '<button class="ok">&#10003;</button>'+
    '</div>');
    $.imageResizer=function(){
        if(Uint8Array&&HTMLCanvasElement&&atob&&Blob){

        }else{
            return false;
        }
        var resizer=tmp.clone();
        resizer.image=resizer.find('img')[0];
        resizer.frames=resizer.find('.frames');
        resizer.okButton=resizer.find('button.ok');
        resizer.frames.offset={
            top:0,
            left:0
        };

        resizer.okButton.click(function(){
            resizer.clipImage();
        });
        resizer.clipImage=function(){
            var nh=this.image.naturalHeight,
                    nw=this.image.naturalWidth,
                    size=nw>nh?nh:nw;

            size=size>1000?1000:size;

            var canvas=$('<canvas width="'+size+'" height="'+size+'"></canvas>')[0],
                    ctx=canvas.getContext('2d'),
                    scale=nw/this.offset.width,
                    x=this.frames.offset.left*scale,
                    y=this.frames.offset.top*scale,
                    w=this.frames.offset.size*scale,
                    h=this.frames.offset.size*scale;

            ctx.drawImage(this.image,x,y,w,h,0,0,size,size);
            var src=canvas.toDataURL();
            this.canvas=canvas;
            this.append(canvas);
            this.addClass('uploading');
            this.removeClass('have-img');

            src=src.split(',')[1];
            if(!src)return this.doneCallback(null);
            src=window.atob(src);

            var ia = new Uint8Array(src.length);
            for (var i = 0; i < src.length; i++) {
                ia[i] = src.charCodeAt(i);
            };

            this.doneCallback(new Blob([ia], {type:"image/png"}));
        };

        resizer.resize=function(file,done){
            this.reset();
            this.doneCallback=done;
            this.setFrameSize(0);
            this.frames.css({
                top:0,
                left:0
            });
            var reader=new FileReader();
            reader.onload=function(){
                resizer.image.src=reader.result;
                reader=null;
                resizer.addClass('have-img');
                resizer.setFrames();
            };
            reader.readAsDataURL(file);
        };

        resizer.reset=function(){
            this.image.src='';
            this.removeClass('have-img');
            this.removeClass('uploading');
            this.find('canvas').detach();
        };

        resizer.setFrameSize=function(size){
            this.frames.offset.size=size;
            return this.frames.css({
                width:size+'px',
                height:size+'px'
            });
        };

        resizer.getDefaultSize=function(){
            var width=this.find(".inner").width(),
                    height=this.find(".inner").height();
            this.offset={
                width:width,
                height:height
            };
            console.log(this.offset)
            return 100;//width>height?height:width;  //这里设置的是选择框的长和高
        };

        resizer.moveFrames=function(offset){
            var x=offset.x,
                    y=offset.y,
                    top=this.frames.offset.top,
                    left=this.frames.offset.left,
                    size=this.frames.offset.size,
                    width=this.offset.width,
                    height=this.offset.height;

            if(x+size+left>width){
                x=width-size;
            }else{
                x=x+left;
            };

            if(y+size+top>height){
                y=height-size;
            }else{
                y=y+top;
            };
            x=x<0?0:x;
            y=y<0?0:y;
            this.frames.css({
                top:y+'px',
                left:x+'px'
            });

            this.frames.offset.top=y;
            this.frames.offset.left=x;
        };
        (function(){
            var time;
            function setFrames(){
                var size=resizer.getDefaultSize();
                resizer.setFrameSize(size);
            };

            window.onresize=function(){
                clearTimeout(time)
                time=setTimeout(function(){
                    setFrames();
                },1000);
            };

            resizer.setFrames=setFrames;
        })();

        (function(){
            var lastPoint=null;
            function getOffset(event){
                event=event.originalEvent;
                var x,y;
                if(event.touches){
                    var touch=event.touches[0];
                    x=touch.clientX;
                    y=touch.clientY;
                }else{
                    x=event.clientX;
                    y=event.clientY;
                }

                if(!lastPoint){
                    lastPoint={
                        x:x,
                        y:y
                    };
                };

                var offset={
                    x:x-lastPoint.x,
                    y:y-lastPoint.y
                }
                lastPoint={
                    x:x,
                    y:y
                };
                return offset;
            };
            resizer.frames.on('touchstart mousedown',function(event){
                getOffset(event);
            });
            resizer.frames.on('touchmove mousemove',function(event){
                if(!lastPoint)return;
                var offset=getOffset(event);
                resizer.moveFrames(offset);
            });
            resizer.frames.on('touchend mouseup',function(event){
                lastPoint=null;
            });
        })();
        return resizer;
    };
    var resizer=$.imageResizer(),
            resizedImage;

    if(!resizer){
        resizer=$("<p>Your browser doesn't support these feature:</p><ul><li>canvas</li><li>Blob</li><li>Uint8Array</li><li>FormData</li><li>atob</li></ul>")
    };

    $('.container').append(resizer);

    $('input').change(function(event){
        var file=this.files[0];
        resizer.resize(file,function(file){
            resizedImage=file;
        });
    });

    $('button.submit').click(function(){
        var url=$('input.url').val();
        if(!url||!resizedFile)return;
        var fd=new FormData();
        fd.append('file',resizedFile);
        $.ajax({
            type:'POST',
            url:url,
            data:fd
        });
    });
});//]]>
</script>
</head>
<body>
<input type="file" accept="images/*">
<input class="url" type="url" placeholder="url">
<div class="container"><div class="resizer"><div class="inner"><img><div class="frames"></div></div><button class="ok">✓</button></div></div>
<button class="submit">Submit</button>
</body>
</html>
