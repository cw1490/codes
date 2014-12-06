```php
<?php   
/**   
* Blacklist   
*   
*  一个简单的CI黑名单过滤库   
*   
* @package      Blacklist   
* @version      1.0   
* @author       JayJun <jayjun0805@sina.com>   
* @copyright    2012 JayJun   
*   
          
class Blacklist   
{   
    /**   
     * 定义需要过滤的词语   
     *   
     * @access  protected   
     * @var     array   
     */
    protected $_words = array();   
          
    /**   
     *   
     * @access protected   
     * @var     bool   
     */
    protected $_blocked = FALSE;   
          
    /**   
     *   
     * @access public   
     * @var     array   
     */
    public $target_text = array();   
          
    /**   
     * Construct   
     *   
     * @access public   
     * @param   array  $config   
     * @var     array   
     */
    public function __construct($config)   
    {   
        if (!empty($config))   
        {   
            foreach ($config as $key => $val)   
            {   
                if (isset($this->{'_' . $key}))   
                {   
                    $val = ! empty($val) ? is_array($val) ? $val : array($val) : array();      
                    $this->{'_' . $key} = $val;   
                }   
            }   
        }   
    }   
          
    // --------------------------------------------------------------------   
          
    /**   
     *   
     * @access public    
     * @return bool    
     */
    public function is_blocked()   
    {   
        return $this->_blocked;   
    }   
          
    // --------------------------------------------------------------------   
          
    /**   
     * 添加新的词语到黑名单中   
     *   
     * @access  public    
     * @param   mixed  array|string    
     * @return  object $this   
     */
    public function add_word($words)   
    {   
        $words = ! is_array($words) ? array($words) : $words;   
          
        $this->_words = array_merge($this->_words, $words);   
          
        return $this;   
    }   
          
    // --------------------------------------------------------------------   
          
    /**   
     * 检查是否包含禁用的词语   
     *   
     * @access public    
     * @param  string  检查$texts   
     * @return object  $this   
     */
    public function check_text($texts)   
    {   
        $this->_set_target_text($texts);   
          
        foreach ($this->_words as $word)    
        {   
            if (stripos($this->target_text, $word) !== false)    
            {   
                $this->_blocked = TRUE;   
          
                log_message('debug', "发现禁用的关键词: '$word' 存在文本中！.");   
            }   
        }   
          
        return $this;   
    }   
          
    // --------------------------------------------------------------------   
          
    /**   
     * 替换禁用的词语   
     *   
     * @access  public   
     * @param   mixed  array|string 提供的一段需要过滤的字符串或数组   
     * @param   string 提供新的字符串替换过滤的词语，默认为*   
     * @return  mixed  array|string    
     */
    public function replace($text, $fill = '*')   
    {   
        // 如果$text是数组   
        if (is_array($text) && !empty($text))   
        {   
            $result = array();   
          
            foreach ($text as $t)    
            {   
                $result[] = $this->replace($t, $fill);   
            }   
          
            return $result;   
        }    
        // 如果$text是字符串   
        else
        {   
            foreach ($this->_words as $word)    
            {   
                if (stripos($text, $word) !== false)    
                {   
                    $replacement = implode('', array_fill(0, iconv_strlen($word, 'UTF-8'), $fill));   
                    $result = str_ireplace($word, $replacement, $text);   
                    $text = $result;   
                }   
            }   
            return $result;   
        }   
          
        return;   
    }   
          
    // --------------------------------------------------------------------   
          
    /**   
     * Set the target text block   
     *   
     * @access  private    
     * @param   mixed  array|string   
     * @return  object $this   
     */
    private function _set_target_text($texts)   
    {   
        $texts = ! is_array($texts) ? array($texts) : $texts;   
          
        $this->target_text = implode("\n", $texts);   
    }   
}   
          
/* End of file Blacklist.php */
```
将以上写好的Blacklist.php文件放到application/libraries目录下，接着需要在初始化黑名单过滤类时传递参数，这里可以传递存于配置文件中的参数，只需简单的建立一个与黑名单过滤类相同的config文件,并保存在 application/config/ 文件夹中即可，这样我们在application/config下将建一个名为blacklist.php的文件，该文件中代码如下：
```php
<?php  
   $config['words'] = array('SB', 'FUCK', '共产党', '法轮功', '迷魂药', '代开发票');  
?>
```
接着我们在就可以在控制器中调用了，例如我们新建一个名为message.php的文件，代码如下：
```php
<?php  
class Message extends CI_Controller {  
   //构造函数  
    public function  __construct()   
    {  
        parent::__construct();  
        //加载CI黑名单过滤类库，初始化时传递的是存于配置文件中的参数  
         $this->load->library('blacklist');  
    }  
           
    public function index()  
    {  
        $str = 'FUCK you!';  
        $this->blacklist->check_text($str)->is_blocked();  
        $this->blacklist->replace($str, '@');   
        echo $str;  //此时打印结果应该为'@@@@ you!'  
    }  
}
```
