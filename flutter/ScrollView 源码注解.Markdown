# ScrollView 源码注解	

```dart
abstract class ScrollView extends StatelessWidget {
  const ScrollView({
    super.key,
    this.scrollDirection = Axis.vertical,
    this.reverse = false,
    this.controller,
    this.primary,
    ScrollPhysics? physics,
    this.scrollBehavior,
    this.shrinkWrap = false,
    this.center,
    this.anchor = 0.0,
    this.cacheExtent,
    this.semanticChildCount,
    this.dragStartBehavior = DragStartBehavior.start,
    this.keyboardDismissBehavior = ScrollViewKeyboardDismissBehavior.manual,
    this.restorationId,
    this.clipBehavior = Clip.hardEdge,
  })
      : assert(scrollDirection != null),
        assert(reverse != null),
        assert(shrinkWrap != null),
        assert(dragStartBehavior != null),
        assert(clipBehavior != null),
        assert(
        !(controller != null && (primary ?? false)),
        'Primary ScrollViews obtain their ScrollController via inheritance '
            'from a PrimaryScrollController widget. You cannot both set primary to '
            'true and pass an explicit controller.',
        ),
        assert(!shrinkWrap || center == null),
        assert(anchor != null),
        assert(anchor >= 0.0 && anchor <= 1.0),
        assert(semanticChildCount == null || semanticChildCount >= 0),
        physics = physics ?? ((primary ?? false) ||
            (primary == null && controller == null &&
                identical(scrollDirection, Axis.vertical))
            ? const AlwaysScrollableScrollPhysics()
            : null);


  ///用于指定滚动视图的滚动方向。
  ///[Axis.vertical] 默认值 垂直滚动
  ///[Axis.horizontal] 水平滚动
  final Axis scrollDirection;

  ///用于确定滚动视图是否以阅读方向反向滚动
  final bool reverse;

  ///滚动控制器对象，用于控制滚动视图的位置和行为
  final ScrollController? controller;

  ///用于确定是否将此滚动视图设置为与父级[PrimaryScrollController]关联的主要滚动视图
  ///为[true]时，即便内容不足以支持滚动 用户也可以滚动视图
  final bool? primary;

  ///定义滚动视图的物理行为，以及用户如何与滚动视图交互
  final ScrollPhysics? physics;

  ///滚动视图中定制行为和外观
  final ScrollBehavior? scrollBehavior;

  ///*************///
  ///[ScrollBehavior] 定制行为和外观
  ///[ScrollPhysics] 定义滚动的物理行为
  ///可以配合使用
  ///*************///

  ///用于确定滚动视图在scrollDirection（滚动方向）中的大小是否应该根据视图内容的大小来确定
  ///true:控件实际大小会与内容一致
  ///false:控件会尽可能大
  final bool shrinkWrap;

  ///用于在自定义滚动视图（CustomScrollView）中指定一个子元素作为生长方向的"中心"点
  final Key? center;

  ///用于指定滚动视图中的零滚动偏移的相对位置
  ///取值在[0.0-1.0]之间
  ///如果是 0.0 那么是在初始位置
  ///如果是 1.0 那么就是在结束位置
  final double anchor;

  ///用于控制视图的缓存区域大小
  ///可用于在滚动方向上更多地缓存 item （用户不可见部分，即提前渲染缓存）
  ///但是如果设置过大容易造成新能损耗
  final double? cacheExtent;

  ///当前情景下起作用的 item 数量
  ///
  final int? semanticChildCount;

  /// 用于确定在用户开始拖动滚动视图时的行为方式
  ///
  final DragStartBehavior dragStartBehavior;

  ///用于确定滚动视图如何自动关闭键盘
  final ScrollViewKeyboardDismissBehavior keyboardDismissBehavior;

  ///[restorationId]是一个用于在应用程序中实现状态恢复的机制的属性。
  ///状态恢复允许你保存和还原item的状态，以便在应用程序关闭和重新启动时保持用户界面的连贯性。
  final String? restorationId;

  ///针对超出视图部分的裁切行为
  final Clip clipBehavior;


  ///获取视图滚动的轴方向
  @protected
  AxisDirection getDirection(BuildContext context) {
    return getAxisDirectionFromAxisReverseAndDirectionality(
        context, scrollDirection, reverse);
  }

  ///获取构建的 slivers 列表
  @protected
  List<Widget> buildSlivers(BuildContext context);


  ///构建滚动视图里面的可见部分
  ///注意 这里根据[shrinkWrap]的不同有不同的 widget 返回
  @protected
  Widget buildViewport(BuildContext context,
      ViewportOffset offset,
      AxisDirection axisDirection,
      List<Widget> slivers,) {
    assert(() {
      switch (axisDirection) {
        case AxisDirection.up:
        case AxisDirection.down:
          return debugCheckHasDirectionality(
            context,
            why: 'to determine the cross-axis direction of the scroll view',
            hint: 'Vertical scroll views create Viewport widgets that try to determine their cross axis direction '
                'from the ambient Directionality.',
          );
        case AxisDirection.left:
        case AxisDirection.right:
          return true;
      }
    }());
    if (shrinkWrap) {
      return ShrinkWrappingViewport(
        axisDirection: axisDirection,
        offset: offset,
        slivers: slivers,
        clipBehavior: clipBehavior,
      );
    }
    return Viewport(
      axisDirection: axisDirection,
      offset: offset,
      slivers: slivers,
      cacheExtent: cacheExtent,
      center: center,
      anchor: anchor,
      clipBehavior: clipBehavior,
    );
  }

  @override
  Widget build(BuildContext context) {
    final List<Widget> slivers = buildSlivers(context);
    final AxisDirection axisDirection = getDirection(context);

    ///[PrimaryScrollController.shouldInherit(context, scrollDirection)]
    ///检查当前的[PrimaryScrollController]是否自动继承[Axis]------区分各个平台
    ///
    ///
    /// ************************************
    ///如果没有设置[primary]
    ///那就找它parents，看是否继承
    final bool effectivePrimary = primary
        ?? controller == null &&
            PrimaryScrollController.shouldInherit(context, scrollDirection);

    /// 如果[effectivePrimary]
    /// true: 使用上下文的滑动控制器
    /// false： 使用自定义的滑动控制器
    final ScrollController? scrollController = effectivePrimary
        ? PrimaryScrollController.maybeOf(context)
        : controller;


    ///根据传入的参数生成对应的[Scrollable]
    final Scrollable scrollable = Scrollable(
      dragStartBehavior: dragStartBehavior,
      axisDirection: axisDirection,
      controller: scrollController,
      physics: physics,
      scrollBehavior: scrollBehavior,
      semanticChildCount: semanticChildCount,
      restorationId: restorationId,
      viewportBuilder: (BuildContext context, ViewportOffset offset) {
        return buildViewport(context, offset, axisDirection, slivers);
      },
      clipBehavior: clipBehavior,
    );

    ///根据[effectivePrimary]以及是否自定义了[ScrollController]生成不同的Scrollable
    ///如果我定义了[ScrollController] 但是我又希望[effectivePrimary]不受主控制器控制
    ///那么就使用[PrimaryScrollController.none(child: scrollable)]包裹它
    ///
    /// 反之
    ///
    /// 就正常使用
    final Widget scrollableResult = effectivePrimary && scrollController != null
    // Further descendant ScrollViews will not inherit the same PrimaryScrollController
        ? PrimaryScrollController.none(child: scrollable)
        : scrollable;

    ///如果我要求我的键盘在滑动的时候被关闭掉
    ///那么就添加一个监听
    ///
    /// 反之
    ///
    /// 直接返回
    if (keyboardDismissBehavior == ScrollViewKeyboardDismissBehavior.onDrag) {
      return NotificationListener<ScrollUpdateNotification>(
        child: scrollableResult,
        onNotification: (ScrollUpdateNotification notification) {
          final FocusScopeNode focusScope = FocusScope.of(context);
          if (notification.dragDetails != null && focusScope.hasFocus) {
            focusScope.unfocus();
          }
          return false;
        },
      );
    } else {
      return scrollableResult;
    }
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(EnumProperty<Axis>('scrollDirection', scrollDirection));
    properties.add(FlagProperty(
        'reverse', value: reverse, ifTrue: 'reversed', showName: true));
    properties.add(DiagnosticsProperty<ScrollController>(
        'controller', controller, showName: false, defaultValue: null));
    properties.add(FlagProperty('primary', value: primary,
        ifTrue: 'using primary controller',
        showName: true));
    properties.add(DiagnosticsProperty<ScrollPhysics>(
        'physics', physics, showName: false, defaultValue: null));
    properties.add(FlagProperty('shrinkWrap', value: shrinkWrap,
        ifTrue: 'shrink-wrapping',
        showName: true));
  }
}

```

