---
layout: post
title: Custom drawing in QML (using the GPU)
date: 2020-01-29 11:04
category: 
author: 
tags: [qt, qml, c++]
summary: 
image: qt-logo-big.png
---

In my project [Freeside Raytracer](https://github.com/tobiasmarciszko/qt_raytracer_challenge) I am creating a wireframe representation of the world to allow for manipulating the scene and modifying objects. To do this, I created a simple "world to screen" conversion of the points in the world, to x and y coordinates in the image to be displayed. I decided to use a QImage as a framebuffer backend which allows me to use the QPainter API for drawing. I started using direct manipulation of pixels via a pointer to the data, but decided to move to the QPainter API instead, for easily drawing lines and polygons. 

The real question then arose how to pass this QImage to QML (since I'm using QML as the UI layer). 

It seems there are numerous ways of passing the image data to QML. The first approach I started off using is to create a custom QML item, derived from QQuickPaintedItem: 

```cpp
class ImageItem : public QQuickPaintedItem
{
    Q_OBJECT
public:
    explicit ImageItem(QQuickItem *parent = nullptr);
    Q_INVOKABLE void setImage(const QImage &image);
    void paint(QPainter *painter) override;

private:
    QImage m_image;
};
```

This item is then used in QML where I need this item to be, e.g: 

```qml
ImageItem {
    id: liveImageItem2
    anchors.right: parent.right
    anchors.top: parent.top

    width: middleRectangle.viewportWidth
    height: middleRectangle.viewportheight

    visible: !settings.fullscreenEnabled

    Text {
        text: "Left"
        anchors.left: parent.left
        anchors.top: parent.top
    }
}
```

And so, whenever the raytracer backend sends a signal to the UI that a wireframe image is ready, the ImageItem is updated with the new image, which in turn is drawn on the next paint() call:

```cpp
void ImageItem::paint(QPainter *painter)
{
    painter->drawImage(0,0, m_image);
}

void ImageItem::setImage(const QImage &image)
{
    m_image = image;
    update();
}
```

So far so good. However, there are never such thing as a silver bullet. Since the drawing using QPainter operates on a QImage, the software rasterizer is used as opposed to using the GPU for drawing. The same thing goes for the QPainter calls inside the QQuickPaintedItem. The call to paint() will also use a software rasterizer. So nothing what I am doing here is done on the GPU. 

That's fine, as long as I'm not pumping the custom item with frames 60 times per second. Luckily I'm not doing that :)

Another option would be to use a QQuickItem derived class instead of QQuickPaintedItem. This is definitely one step close to the hardware, but we end up in another abstraction instead, namely the SceneGraph which QML is based on: 

```cpp
QSGNode *ImageItem::updatePaintNode(
    QSGNode *oldNode, QQuickItem::UpdatePaintNodeData *updatePaintNodeData) {

    auto node = dynamic_cast<QSGSimpleTextureNode *>(oldNode);

    if (!node) {
        node = new QSGSimpleTextureNode();
    }

    QSGTexture *texture = window()->createTextureFromImage(m_image, QQuickWindow::TextureIsOpaque);
    node->setOwnsTexture(true);
    node->setRect(boundingRect());
    node->markDirty(QSGNode::DirtyForceUpdate);
    node->setTexture(texture);
    return node;
}
``` 

This example will use the GPU for drawing, but we still have to convert the QImage into a texture that we can use for rendering. Here we paint a node in the SceneGraph instead of "paint()" and create the appropriate nodes, materials and textures for the custom item. 

Going even one step further down is to avoid drawing to a QImage, but instead do all drawing inside the updatePaintNode call. 

This means that the wireframe creation has to happen here, but we can potentially pass a data structure such as the world with all objects, or a list of lines, or a list of polygons, to draw. 

This means that we could potentially end up with something like this: 

```cpp

QSGNode *ImageItem::updatePaintNode(
    QSGNode *oldNode, QQuickItem::UpdatePaintNodeData *updatePaintNodeData) {

    if (!oldNode) {
        oldNode = new QSGNode;
    }

    oldNode->removeAllChildNodes();

    for (const QLine& line: m_lines) {

        auto childNode = new QSGGeometryNode;

        auto geometry = new QSGGeometry(QSGGeometry::defaultAttributes_Point2D(), 2);
        geometry->setLineWidth(2);
        geometry->setDrawingMode(QSGGeometry::DrawLineStrip);
        childNode->setGeometry(geometry);
        childNode->setFlag(QSGNode::OwnsGeometry);

        auto material = new QSGFlatColorMaterial;
        material->setColor(QColor(255, 0, 0));
        childNode->setMaterial(material);
        childNode->setFlag(QSGNode::OwnsMaterial);

        QSGGeometry::Point2D *vertices = geometry->vertexDataAsPoint2D();

        vertices[0].set(line.x1(), line.y1());
        vertices[1].set(line.x2(), line.y2());

        oldNode->appendChildNode(childNode);
    }
    return oldNode;
    }
```

In this example, we have a parent node which doesn't contain any geometry, but then add new child nodes with geometry (lines) for all the lines we have passed to the custom item. 

This exact example isn't very well tested and optimized. There are object allocations for the geometry and material for every line that is to be drawn. I don't think that's a good thing and we can probably manage that memory in our item object instead. The geometry and material are the same for all the lines, but updated with different coordinates and colors for every paint cycle. 

The last option would be to potentially use a QML Canvas element and draw the primitives there based on the world as input or lines and polygons such as in the custom item case. 

I have yet to come to a conclusion on what the "best" way is to pass or draw custom items in an efficient way from C++ to QML. Especially if you need a "game loop" or framebuffer pump which allows for full GPU optimization. 

I even wrote a question in the [Qt forum](https://forum.qt.io/topic/111022/pixel-framebuffer-data-in-c-to-qml-application-using-the-gpu-as-much-as-possible) but when writing this post I haven't got a single answer to this question. Maybe I'm the only one experimenting on the fringe of these frameworks. 

If you any suggestion, please drop me a DM on twitter or on GitHub. 

That's all folks! :)
