# TQRichTextViewTest
**富文本控件实验**
##文件目录结构
1. EmojiLmage：文件夹存放的是表情图片资源。
2. TQRichTextBaseRun：这个是特殊文本元素的抽象对象。这里定义了需要子类实现方法和属性。URLRun，ImageRun 等都是从这个继承。
3. TQRichTextURLRun：这个是用来支持URL文本类型的文本元素对象。
4. TQRichTextImageRun：这个是图片类型的文本元素对象。比如表情文本其实是要绘制图片到文本中间。后面的EmojiRun就是派生自这个对象。
5. TQRichTextEmojiRun：这个是表情文本元素对象，用来识别表情字符串替换成表情图片工作。
6. TQRichTextView：文本显示所用到的视图

##富文本解析过程

1. 取得要现实富文本的文本字符串
2. 我们需要对这个字符串进行解析
3. 绘制，它可以将文本全部给CoreText，然后它会给你绘制好。包括换行等。你只需要在表情的地方流出空白，然后再绘制上图片

得到文本后，我们对文本进行解析。具体解析算法呢，都是写在对应的__RichTextxxxRun__中的。比如表情解析就是在TQRichTextEmojiRun 中。url同理。在解析的过程中，比如解析表情。匹配到了表情，就生成一个__TQRichTextEmojiRun__ 对象，这个对象记录了这个表情文本的__位置__。__原始字符__等信息。把根据生成的对象放入__richTextRunsArray__这么一个数组中。用于文字渲染完后根据这个对象渲染表情图片。因为都是继承baseRun来的。所以，解析出来的不管是表情，还是url什么，都会统一执行渲染方法。当我们解析完，得到了特殊文本的run数组对象。我们就绘制文本。然后填补表情等。这些就是在__drawRect__这个方法里面执行的了

__BaseRun__ 里面抽象的2个方法

	```objc
	
	//-- 替换基础文本
	- (void)replaceTextWithAttributedString:(NSMutableAttributedString*) attributedString
	{
    	[attributedString 	addAttribute:@"TQRichTextAttribute" value:self 	range:self.range];
	}

	//-- 绘制内容
	- (BOOL)drawRunWithRect:(CGRect)rect
	{
    	return NO;
	}
	
	```
1. replaceTextWithAttributedString 这个方法，是用来替换文本的。说是替换，其实有2中情况。如果文本单元是一个表情的话。我需要将原先的字符串用一个空格代替 ，然后预留出画表情的位置。绘制表情图片在图层在比如文本单元是一个URL的话。其实这里没有替换以前的URL，只是设置了属性字符串。在URL这一段，让他蓝色显示。

2. drawRunWithRect
这个方法是绘制文本单元。和上面类似。如果表情文本单元，这个就要负责回事图片到图层。如果是URL，这个方法直接返回NO 就好了。告诉绘制的时候。这个方法什么都没有做。（这跟后面做触摸响应时间时候。获得正确的点击区域判断有关系。）