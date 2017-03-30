---
title: NSUserDefaults错误
date: 2017-03-30 17:58:35
tags: IOS
---

```
Attempt to insert non-property list object (
    "<NovelChapterModel: 0x170038160>",
    "<NovelChapterModel: 0x170036480>",
    "<NovelChapterModel: 0x1700378c0>",
    "<NovelChapterModel: 0x170035f20>",
    "<NovelChapterModel: 0x170036620>",
    "<NovelChapterModel: 0x170038bc0>",
    "<NovelChapterModel: 0x170038600>",
    "<NovelChapterModel: 0x1700357a0>",
    "<NovelChapterModel: 0x170039160>",
    "<NovelChapterModel: 0x170033d60>",
    "<NovelChapterModel: 0x170036380>",
    "<NovelChapterModel: 0x170037dc0>",
    "<NovelChapterModel: 0x1700384c0>",
    "<NovelChapterModel: 0x1700385a0>",
    "<NovelChapterModel: 0x170037b40>",
    "<NovelChapterModel: 0x170038100>",
    "<NovelChapterModel: 0x170035d00>",
    "<NovelChapterModel: 0x170033ea0>",
    "<NovelChapterModel: 0x170033ca0>",
    "<NovelChapterModel: 0x170036100>",
    "<NovelChapterModel: 0x170037b60>",
    "<NovelChapterModel: 0x170037f40>",
    "<NovelCh
libc++abi.dylib: terminating with uncaught exception of type NSException
```

这种错误的原因是插入了不识别的数据类型，NSUserDefaults支持的数据类型有NSString、 NSNumber、NSDate、 NSArray、NSDictionary、BOOL、NSInteger、NSFloat等系统定义的数据类型。自定义的类型需要转成NSData再存入。

1.修改NovelChapterModel,实现NSCoding接口

```objc
@interface NovelChapterModel : NSObject<NSCoding>
@property NSString *chapterName;
@property int chapter;
@end
```
```objc
@implementation NovelChapterModel
-(void)encodeWithCoder:(NSCoder *)aCoder
{
    [aCoder encodeObject:_chapterName forKey:@"chapterName"];
    [aCoder encodeObject: [NSNumber numberWithInt:_chapter] forKey:@"chapter"];
}
-(id)initWithCoder:(NSCoder *)aDecoder
{
    if (self = [super init]) {
        _chapterName = [aDecoder decodeObjectForKey:@"chapterName"];
        _chapter = [[aDecoder decodeObjectForKey:@"chapter"] intValue];
    }
    return self;
}
@end
```

2.序列化存入

```objc
NSData *nsData = [NSKeyedArchiver archivedDataWithRootObject:chapterArray];
[defaults setObject:nsData forKey:key];
```

3.反序列化取出

```objc
NSUserDefaults *defaults =[NSUserDefaults standardUserDefaults];
NSData *chapterFromCache =  [defaults objectForKey:key];
NSMutableArray *chapterArray;
if(chapterFromCache!=nil)
{
    chapterArray = [NSKeyedUnarchiver unarchiveObjectWithData:chapterFromCache];
}
```