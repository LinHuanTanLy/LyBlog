# Viewport 源码解析

```dart
class Viewport extends MultiChildRenderObjectWidget {
  Viewport({
    super.key,
    this.axisDirection = AxisDirection.down,
    this.crossAxisDirection,
    this.anchor = 0.0,
    required this.offset,
    this.center,
    this.cacheExtent,
    this.cacheExtentStyle = CacheExtentStyle.pixel,
    this.clipBehavior = Clip.hardEdge,
    List<Widget> slivers = const <Widget>[],
  })
      : assert(offset != null),
        assert(slivers != null),
        assert(center == null || slivers
            .where((Widget child) => child.key == center)
            .length == 1),
        assert(cacheExtentStyle != null),
        assert(cacheExtentStyle != CacheExtentStyle.viewport ||
            cacheExtent != null),
        assert(clipBehavior != null),
        super(children: slivers);

  ///滚动布局滚动的方向
  final AxisDirection axisDirection;

  ///item 在横轴上面的方向
  final AxisDirection? crossAxisDirection;

  ///用于指定滚动视图中的零滚动偏移的相对位置
  ///取值在[0.0-1.0]之间
  ///如果是 0.0 那么是在初始位置
  ///如果是 1.0 那么就是在结束位置
  final double anchor;


  ///窗口的偏移量
  final ViewportOffset offset;

  ///用于在自定义滚动视图（CustomScrollView）中指定一个子元素作为生长方向的"中心"点
  final Key? center;

  ///用于控制视图的缓存区域大小
  ///可用于在滚动方向上更多地缓存 item （用户不可见部分，即提前渲染缓存）
  ///但是如果设置过大容易造成新能损耗
  final double? cacheExtent;

  ///用于控制视口[Viewport] 的缓存区域[cacheExtent]的模式
  ///[CacheExtentStyle.pixel] 以像素模式
  ///[CacheExtentStyle.viewport】跟当前视图一样
  final CacheExtentStyle cacheExtentStyle;

  ///针对超出视图部分的裁切行为
  final Clip clipBehavior;


  ///获取交叉轴滚动方向
  ///通过交叉轴的方向来获取文本的展示方向
  ///
  /// 如果是[AxisDirection.up]或者[AxisDirection.down]
  /// 会根据[Directionality.of(context)]获取文本排列方向
  /// 最终来确定文本的展示
  static AxisDirection getDefaultCrossAxisDirection(BuildContext context,
      AxisDirection axisDirection) {
    assert(axisDirection != null);
    switch (axisDirection) {
      case AxisDirection.up:
        assert(debugCheckHasDirectionality(
          context,
          why: "to determine the cross-axis direction when the viewport has an 'up' axisDirection",
          alternative: "Alternatively, consider specifying the 'crossAxisDirection' argument on the Viewport.",
        ));
        return textDirectionToAxisDirection(Directionality.of(context));
      case AxisDirection.right:
        return AxisDirection.down;
      case AxisDirection.down:
        assert(debugCheckHasDirectionality(
          context,
          why: "to determine the cross-axis direction when the viewport has a 'down' axisDirection",
          alternative: "Alternatively, consider specifying the 'crossAxisDirection' argument on the Viewport.",
        ));
        return textDirectionToAxisDirection(Directionality.of(context));
      case AxisDirection.left:
        return AxisDirection.down;
    }
  }

  ///创建对应的 renderobject
  @override
  RenderViewport createRenderObject(BuildContext context) {
    return RenderViewport(
      axisDirection: axisDirection,
      crossAxisDirection: crossAxisDirection ??
          Viewport.getDefaultCrossAxisDirection(context, axisDirection),
      anchor: anchor,
      offset: offset,
      cacheExtent: cacheExtent,
      cacheExtentStyle: cacheExtentStyle,
      clipBehavior: clipBehavior,
    );
  }

  @override
  void updateRenderObject(BuildContext context, RenderViewport renderObject) {
    renderObject
      ..axisDirection = axisDirection
      ..crossAxisDirection = crossAxisDirection ??
          Viewport.getDefaultCrossAxisDirection(context, axisDirection)
      ..anchor = anchor
      ..offset = offset
      ..cacheExtent = cacheExtent
      ..cacheExtentStyle = cacheExtentStyle
      ..clipBehavior = clipBehavior;
  }

  @override
  MultiChildRenderObjectElement createElement() => _ViewportElement(this);

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(EnumProperty<AxisDirection>('axisDirection', axisDirection));
    properties.add(EnumProperty<AxisDirection>(
        'crossAxisDirection', crossAxisDirection, defaultValue: null));
    properties.add(DoubleProperty('anchor', anchor));
    properties.add(DiagnosticsProperty<ViewportOffset>('offset', offset));
    if (center != null) {
      properties.add(DiagnosticsProperty<Key>('center', center));
    } else if (children.isNotEmpty && children.first.key != null) {
      properties.add(DiagnosticsProperty<Key>(
          'center', children.first.key, tooltip: 'implicit'));
    }
    properties.add(DiagnosticsProperty<double>('cacheExtent', cacheExtent));
    properties.add(DiagnosticsProperty<CacheExtentStyle>(
        'cacheExtentStyle', cacheExtentStyle));
  }
}

///Viewport 对应的 Element
class _ViewportElement extends MultiChildRenderObjectElement
      with NotifiableElementMixin, ViewportElementMixin {
  /// Creates an element that uses the given widget as its configuration.
  _ViewportElement(Viewport super.widget);

  ///当前是否正在 mount 或者更新
  bool _doingMountOrUpdate = false;

  ///center 的 key 对应的卡槽的 index
  int? _centerSlotIndex;

  @override
  RenderViewport get renderObject => super.renderObject as RenderViewport;

  @override
  void mount(Element? parent, Object? newSlot) {
    assert(!_doingMountOrUpdate);
    _doingMountOrUpdate = true;
    super.mount(parent, newSlot);
    _updateCenter();
    assert(_doingMountOrUpdate);
    _doingMountOrUpdate = false;
  }

  @override
  void update(MultiChildRenderObjectWidget newWidget) {
    assert(!_doingMountOrUpdate);
    _doingMountOrUpdate = true;
    super.update(newWidget);
    _updateCenter();
    assert(_doingMountOrUpdate);
    _doingMountOrUpdate = false;
  }

  ///更新 center 的 key 对应的数据
  void _updateCenter() {
    final Viewport viewport = widget as Viewport;
    if (viewport.center != null) {
      ///如果之前已经制定了 center 了
      int elementIndex = 0;

      ///遍历拿到 childrens 里面对应的 element
      ///把它对应的 renderobject 赋值给对应的 center key element
      ///并且记录它的下标为_centerSlotIndex
      for (final Element e in children) {
        if (e.widget.key == viewport.center) {
          renderObject.center = e.renderObject as RenderSliver?;
          break;
        }
        elementIndex++;
      }
      assert(elementIndex < children.length);
      _centerSlotIndex = elementIndex;
    } else if (children.isNotEmpty) {
      ///如果没有 centerkey 拿 children 的第一个 item 便是默认
      ///把它的 renderObject 赋值给当前的 renderObject.center
      ///并且记录它的下标为_centerSlotIndex为 0
      renderObject.center = children.first.renderObject as RenderSliver?;
      _centerSlotIndex = 0;
    } else {
      ///如果当前的 viewport 没有子元素
      ///那么清空 renderObject.center 以及 _centerSlotIndex
      renderObject.center = null;
      _centerSlotIndex = null;
    }
  }

  ///在father 类[MultiChildRenderObjectElement]里面已经做了 insert 操作了
  ///[renderObject.insert(child, after: slot.value?.renderObject);]
  ///这里为什么要重新 for 循环遍历一遍
  ///是因为[slot]可能带有 center 属性
  ///而当列表数据更新的时候，[insertRenderObjectChild]必然会被调用
  ///这个时候需要重新赋值[renderObject.center]以便布局渲染为居中
  @override
  void insertRenderObjectChild(RenderObject child, IndexedSlot<Element?> slot) {
    super.insertRenderObjectChild(child, slot);
    // Once [mount]/[update] are done, the `renderObject.center` will be updated
    // in [_updateCenter].
    if (!_doingMountOrUpdate && slot.index == _centerSlotIndex) {
      renderObject.center = child as RenderSliver?;
    }
  }

  ///移动
  @override
  void moveRenderObjectChild(RenderObject child, IndexedSlot<Element?> oldSlot,
      IndexedSlot<Element?> newSlot) {
    super.moveRenderObjectChild(child, oldSlot, newSlot);
    assert(_doingMountOrUpdate);
  }

  ///移除
  @override
  void removeRenderObjectChild(RenderObject child, Object? slot) {
    super.removeRenderObjectChild(child, slot);
    if (!_doingMountOrUpdate && renderObject.center == child) {
      renderObject.center = null;
    }
  }

  @override
  void debugVisitOnstageChildren(ElementVisitor visitor) {
    children.where((Element e) {
      final RenderSliver renderSliver = e.renderObject! as RenderSliver;
      return renderSliver.geometry!.visible;
    }).forEach(visitor);
  }
}

```

