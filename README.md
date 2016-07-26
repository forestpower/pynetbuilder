## pyNetBuilder
pyNetBuilder is a modular pytonic interface with builtin modules for generating popular caffe networks.

#### Why pynetbuilder?
A neural network is a Directed acyclic graph (DAG) of layers. The caffe [layers](http://caffe.berkeleyvision.org/tutorial/layers.html) and the network is represented using prototxt format. As we go deeper, and add more layers or build more complex DAG's using basic layers, writing the prototxt files becomes tedious. This tool aims to provide a pytonic interface to generate prototxt files. 
 * It provides reusable components in form of ***legos*** from popular networks like inception, resnet, squeezenet etc. 
 * It provides apps which can be used to generate these popular networks for classification and object detection.
 * The complexity module computes the number of parameters and connections or flops of the network generated by the app, which can be used to tweak various network architectures so that you can train networks which suit your memory / runtime needs.
 * We also release some models pretrained on imagenet with few tweaks to the residual network architectures (which also demonstrate how to generate networks using pynetbuilder). Refer [residual network generation] (./models/imagenet/Readme.md) , [SSD generation] (./models/voc2007_ssd/Readme.md) for more details.

#### Dependencies
 1. Caffe (version should include the layer definitions you need for your network)
 2. Google protobuf

#### Design idea
pyNetBuilder builds on top of caffe's [NetSpec](https://github.com/BVLC/caffe/blob/master/python/caffe/net_spec.py#L163) class to stich together a network. NetSpec "provides a way to write nets directly in Python, using a natural,
functional style." [Here](https://github.com/BVLC/caffe/blob/master/examples/pycaffe/caffenet.py) is a basic example of writing nets in caffe. pyNetBuilder is to provide generic wrappers to **attach** blocks of layers to NetSpec in form of **Legos**. 

##### Lego
A [lego](./netbuilder/lego/base.py) is a basic neural network building block. It has:
 * Required parameters - These parameters must be specified while creating a lego, and usually change for each instantiation.
 * Default parameters (which can be overridden). The default parameters can be stored and read from a [config file](./config/default.params).
 * A [attach method](./netbuilder/lego/base.py#L67-L74) which attaches a caffe layer to a netspec object.
 
##### Lego Types:
* BaseLegoFunction: This is a classes which can attach all the core caffe layers using functional style. The default parameters for layers can be added / updated in the [default.config](./config/default.params) file. For example you can attach a convolutional layer to a netspec object as follows:
```python
from lego.base import BaseLegoFunction
conv_params = dict(name='conv1', num_output=64, kernel_size=7,
  use_global_stats='True', pad=3, stride=2)
conv = BaseLegoFunction('Convolution',params).attach(netspec, [netspec.data])
```

* Hybrid - These are combinations of core legos. Example - [ShortcutLego](./netbuilder/lego/hybrid.py#L260) (resnets), [FireLego](./netbuilder/lego/hybrid.py#L86) (squeezenet), [InceptionLego](./netbuilder/lego/hybrid.py#L130) (google).
Example - you can attach a ShortcutLego (residual networks) to a netspec object as follows:
```python
from lego.hybrid import ShortcutLego
params = dict(name='resnet_block', num_output=256,
                          shortcut='identity', main_branch='bottleneck',
                          stride=1, use_global_stats=false,)
block = ShortcutLego(params).attach(netspec, [last_layer])
```

To generate a network, you can pass a network specification through a series of legos and the modules will get attached to the Netspec object.


#### Apps
The apps folder is a collection of python scripts which uses the pynetbuilder modules to create standard caffe network prototxt files from popluar papers. Currently apps for following networks are provided (other contributions are welcome):
 * Cifar 10 training:
   * Resnet - see quick getting started [tutorial](./models/cifar10/Readme.md) and some results here.
 * Imagenet training:
   * Resnet -  We also release caffemodels trained on imagenet with various variants of Residual network architectures. See [Tutorial and results](./models/imagenet/Readme.md) for more details.
   * Squeezenet
   * Alexnet
   * VGGNet
 * Object detection networks - using [SingleShot multibox Detector] (https://github.com/weiliu89/caffe/tree/ssd) - [Tutorial and results](./models/voc2007_ssd/Readme.md)
 
 
#### Contributing to pyNetBuilder
Contributions to pynetbuilder are welcome.
 * If you want to contribute hybrid legos used in your networks please inherit the ```BaseLego``` class and write your hybrid lego inside the [hybrid module](./netbuilder/lego/hybrid.py). If your hybrid legos are more generic (example ssd-for object detection) you can create another module inside ```pynetbuilder.legos.yourlegos``` and add the hybrid lego's inside the module.
 * If you built a different generic app using already existing legos, you can add it to the app folder and contribute the app.
Send a PR and we will review and merge it.


### License
Code licensed under the [BSD 2 clause license] (https://github.com/BVLC/caffe/blob/master/LICENSE). See LICENSE file for terms. 

### Contact
pynetbuilder was written by [Jay Mahadeokar](https://github.com/jay-mahadeokar/) from the Yahoo Vision and ML team. Special thanks to [Jack Culpepper](https://github.com/jackculpepper), [Huy Nguyen](https://github.com/huyng), [Pierre Garrigues](https://github.com/pierreg), [Sachin Farfade](https://github.com/sachinfarfade/), [Clayton Mellina](https://github.com/pumpikano) and other members of the team for inputs and review.
