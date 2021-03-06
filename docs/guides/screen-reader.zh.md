# 屏幕阅读器指南

> 因为每个人都应该有伟大的特征

`react-beautiful-dnd`号飞船，使用伟大的屏幕阅读器支持,在英语中,箱外。如果你只是想入门，那么你就没有什么可做的了。但是,如果你想定制消息,且完全控制。

本指南是为了帮助您创建支持和愉悦用户的消息。屏幕阅读器体验主要关注键盘交互,但是屏幕阅读器用户可以使用任何输入类型(例如鼠标和键盘).

## 音调

在默认消息中,我们选择了友好的语调,在"放下物件"之类的短语中，选择了"您放下了物品"这样的短语。我们知道它是更坏的,但是我们认为代词使它个人化,听起来不像计算机.

选择一个最能，支持你的听众想要做的音调。如果你需要一些灵感和最佳实践指南, 上[https://atlassian.design](https://atlassian.design)看看我们如何与Atlassian的客户沟通.

## 如何控制公告

这个`announce`函数提供给每个`DragDropContext > Hook`功能,可以用来传递自己的屏幕阅读器消息。消息将立即被读出。重要的是**立即**传递消息,这样你的用户有快速反应的时机.

如果你试图控住`announce`函数,之后再调用它,它将无法工作,只会打印警告到控制台。如果尝试为同一事件调用announce两次,则屏幕阅读器将只读取第一个,随后调用的announce，将被忽略并打印警告.

## 覆盖指令

### 第1步:引入拖动物品

当用户`tabs`到一个`Draggable`，我们需要告诉他们如何开始拖拽.我们是通过使用，*拖动控制*的`aria-roledescription`属性完成的.

**默认消息**: "拖动物品"。按**空格键**抬起"

我们告诉用户如下:

-   该物品是可拖动的.
-   如何开启

你不需要在这一点上给出所有的拖动动作指令,让我们等到用户决定再开始拖动.

考虑，将"item-物品"这个词代替一个与你的问题领域相匹配的名词,例如"任务-task"或"问题-issue"。你可能还想把"item"全部删除.

### 步骤2:启动拖动

当用户通过使用`空格`举起`Draggable`，我们想告诉他们很多事情.

**默认消息**: "你已经举起一个物品且在`${start.source.index + 1}`位置上。 使用箭头键移动,再空格就放下,esc就取消.

我们告诉用户如下:

-   他们举起了这个物品.
-   物品处于什么位置
-   如何移动物品

注意我们没有告诉他们，物品处于`1 of x`位置上。 这是因为我们无法访问当前API中的列表大小。就像现在一样,当我们操控虚拟列表时, 要保持API敞亮和功能的明确。在您自己的消息中，随意添加`1 of x`,以及物品的列表.

**消息的其他信息**: "你已经举起一个物品了，处在`${listLength}`长度的`${listName}`列表的第几个`${startPosition}`位置。使用箭头键移动,再空格就放下,esc就取消.

您可以通过`DragDropContext` > `onDragStart`钩方式，控制打印给用户的消息:

```js
onDragStart = (start: DragStart, provided: HookProvided) => {
  provided.announce('My super cool message');
};
```

### 步骤3:拖曳运动

当用户开始拖动时,会出现不同的场景,因此我们将为每个场景，创建不同的消息传递.

我们可以通过`DragDropContext` > `onDragUpdate`钩子，来控制通告.

```js
onDragUpdate = (update: DragUpdate, provided: HookProvided) => {
  provided.announce('Update message');
};
```

#### 场景1.在同一列表中移动

用户已经在同一个列表中，上下(左右)移动,所以我们想告诉用户他们现在处于什么位置.

**默认消息**: "您已经将物品移到了位置`${update.destination.index + 1}`".

考虑包括`${listLength}`，在您的消息传递中.

#### 场景2.搬进不同的列表

用户已经在跨越，移动到一个不同的列表中,所以我们想告诉他们一些事情.

**默认消息**: "您已将物品从`${update.source.droppableId}`列表的`${update.source.index + 1}位置移出到`${update.destination.droppableId}`列表的`${update.destination.index + 1}`"位置.

我们告诉用户如下:

-   他们已经搬到一个新的列表上去了.
-   关于新列表的一些信息
-   他们从什么地方搬来的?
-   他们现在处于什么位置?

在消息中，考虑使用让可拖动的物品呈现一种友好的文本,并包括列表的长度.

**消息的其他信息**: "您已将物品从`${sourceLength}`长度的`${sourceName}`列表的`${lastIndex}`位置移到`${destinationLength}`长度的 `${destinationName}` 列表的 `${newIndex}`位置".

#### 场景3.没有列表移动

您无法使用键盘进行此操作,但却要为此场景留言,以防用户误会此为可拖动的指针.

**默认消息**: "你目前没有拖着一个可放下的区域".

考虑一下如何使这个消息更加友好和清晰.

### 第4步:放下

放下可能有两种方式。要么拖动被取消,要么用户确是放下了拖动项。您可以使用这些控件的`DragDropContext > onDragEnd`钩子，来控制这些事件的消息传递..

#### 场景1.拖曳取消

具有`reason`属性，其可以是`DROP`或`CANCEL`的一个`DropResult`对象。 您可以使用它来宣布取消消息.

```js
onDragEnd = (result: DropResult, provided: HookProvided) => {
  if (result.reason === 'CANCEL') {
    provided.announce('Your cancel message');
    return;
  }
};
```

**默认消息**: "运动取消了. 该项已返回到${RESULTS.SURCE.KED+1 }的起始位置.

我们告诉用户如下:

-   拖拽被取消了
-   物品返回的地方

考虑添加有关列表长度的信息,以及已删除的列表的名称.

**消息的其他信息**: "运动取消了.物品已返回`${listLength}`长度的列表`${listName}`的起始位置`${startPosition}`".

#### 场景2.落户(新位置)

**默认消息**: "你已经放下了这个物品.它已经从`${result.source.index + 1}`位置移动到`${result.destination.index + 1}`"

我们告诉用户如下:

-   他们已经完成了拖拽.
-   物品现在处于什么位置?

#### 场景3.落在家里的列表(原来的位置)

**默认消息**: "你已经放下了这个物品.它已经落在它的起始位置`${result.source.index + 1}`"

我们告诉用户如下:

-   他们已经完成了拖拽.
-   他们把物品放在起始位置
-   起始位置

#### 场景4.落在国外列表上

这个场景的消息传递应该类似于"dropinahomelist",但我们还添加了物品开始的列表以及物品最后的位置.

**默认消息**: "你已经放下了这个物品.它已经从列表`${result.source.droppableId}`的`${result.source.index + 1}`位置移动到列表`${result.destination.droppableId}`的`${result.destination.index + 1}`位置"

考虑一下包括`Droppable`名称，可替代下ID。您可能还希望包含该位置，若`Droppable`是有序的.

#### 场景5.落在没有目的地

您无法使用键盘进行此操作,但值得为此场景留言,以防用户误会有此操作.

**默认消息**: "该物品已被放下,但却不是在一个可放下的位置。该项已返回到${RESULTS.SURCE.KED+1 }的起始位置.

我们告诉用户如下:

-   他们掉到一个不可放下的地方.
-   物品返回的地方

## 这就是所有人

我们希望你觉得这个指南有用。您可以大胆反馈关于您希望包含的场景的建议,或者您可能希望共享自己的默认消息，和让我们进一步增长知识.
