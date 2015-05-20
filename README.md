[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)(RAC)([行为驱动开发](http://www.cocoachina.com/ios/20150518/11853.html))是一个用Objective-C编写，具有函数式和响应式特性的编程框架。
> 很不错的RAC资源教程
> 
> * [nshipster](http://nshipster.com/reactivecocoa/)
> * [reactivecocoa-for-a-better-world](https://github.com/blog/1107-reactivecocoa-for-a-better-world)
> * [MVVM模式](http://objccn.io/issue-13-1/)
> * [ReactiveViewModel](https://github.com/ReactiveCocoa/ReactiveViewModel) 
> * [MVVM-IOS-Example](https://github.com/Machx/MVVM-IOS-Example)
> * [花瓣网李忠：ReactiveCocoa是Cocoa的未来](http://www.infoq.com/cn/news/2014/07/reactiveCocoa-cocoa)
> * [Limboy博客ReactiveCocoa与Functional Reactive Programming](http://limboy.me/ios/2013/06/19/frp-reactivecocoa.html) [说说ReactiveCocoa 2](http://limboy.me/ios/2013/12/27/reactivecocoa-2.html) [ReactiveCocoa2实战](http://limboy.me/ios/2014/06/06/deep-into-reactivecocoa2.html)
> * [南峰子博客：Category: reactivecocoa](http://southpeak.github.io/blog/categories/reactivecocoa/)
> * [sunnyxx博客](http://blog.sunnyxx.com/tags/Reactive%20Cocoa%20Tutorial/)
> * [唐巧博客：ReactiveCocoa - iOS开发的新框架](http://blog.devtang.com/blog/2014/02/11/reactivecocoa-introduction/)

####RACObserve
######KVO监测属性
    [RACObserve(self.username) subscribeNext:^(NSString *newName) {
      NSLog(@"%@", newName);
    }];
######属性值改变KVO监测并赋值界面层  
    RAC(self.userNameLable,text) = RACObserve(self,username)
######过滤条件操作
	[[RACObserve(self, password)
	  filter:^BOOL(NSString *text) {
		return [text length] > 6;
	}]subscribeNext:^(id x) {
		NSLog(@"%@",x);
	}];
######条件符合改变界面
	RAC(self.passwordTextField,backgroundColor) = [RACObserve(self, password)map:^id(NSString *text) {
		return [text length] > 6 ? [UIColor clearColor] : [UIColor yellowColor];
	}];
	RAC(self.usernameTextField,textColor) = [RACObserve(self.usernameTextField, editing) map:^UIColor *(NSNumber *editing) {
		return editing ? [UIColor redColor] : [UIColor blackColor];
	}];
######多个属性值条件改变界面
    RAC(self.signInButton,enabled) = [RACSignal 
    combineLatest:@[RACObserve(self.password), RACObserve(self.passwordConfirmation) ] 
    reduce:^(NSString *password, NSString *passwordConfirm) {
        return @([passwordConfirm isEqualToString:password]);
    }];
####Event
######按钮按下检测
	[[[self.signInButton rac_signalForControlEvents:UIControlEventTouchUpInside]map:^id(id value) {
		return self.signInButton;
	}]subscribeNext:^(id x) {
		NSLog(@"Sign in result: %@", x);
	}];
######按钮按下通过Signal信号传递事件并且执行
	[[[self.signInButton rac_signalForControlEvents:UIControlEventTouchUpInside] flattenMap:^RACStream *(id value) {
		return [self signInSignal];
	}] subscribeNext:^(NSNumber *signedIn) {
		BOOL success = [signedIn boolValue];
		if (success) {
			[self performSegueWithIdentifier:@"signInSuccess" sender:self];
		}
	}];
######ReactiveCocoa每个UIButton下都有一个rac_command属性，按下按钮都会实现一个事件
```
RACCommand *loginCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
   RACSignal *authSignal = [RACSignal empty];
   return authSignal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
			//do login somethings
			return nil;
		}];
	}];
	[loginCommand.executionSignals subscribeNext:^(RACSignal *loginSignal) {
		[loginSignal subscribeCompleted:^{
			NSLog(@"Logged in successfully!");
		}];
}];
self.signInButton.rac_command = loginCommand;
```
```
self.signInButton.rac_command = [[RACCommand alloc] initWithEnabled:validUsernameSignal 
signalBlock:^RACSignal *(id input) {
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            //do login request something
            return nil;
        }];
 }];
    
RACSignal *startSignal = [self.button.rac_command.executionSignals map:^id(id value) {
        return @"Sending Request...";
}];
    
RACSignal *successSignal = [self.button.rac_command.executionSignals switchToLatest];
    
RACSignal *failSignal = [self.button.rac_command.errors map:^id(id value) {
        return @"Request Error";
}];
    
RAC(self.stateLable,text) = [RACSignal merge:@[startSignal, successSignal, failSignal]];
```

####RACSignal
    RACSignal *validUsernameSignal = [self.usernameTextField.rac_textSignal map:^id(NSString *text) {
        return @(text.length > 0); 
    }
    RACSignal *validPasswordSignal = [self.passwordTextField.rac_textSignal map:^id(NSString *text) {
        return @(text.length > 0);
    }];
    
    [validUsernameSignal take:1]subscribeNext:^(id x) {
		NSLog(@"take from 1 time: %@", x);
	 }];
	 
	[validUsernameSignal skip:1]subscribeNext:^(id x) {
		NSLog(@"skip from 1 : %@", x);
	 }];
	 
	[validUsernameSignal takeLast:6]subscribeNext:^(id x) {
		NSLog(@"take from last 6 : %@", x);
	}];
	 
	[[self.usernameTextField.rac_textSignal takeUntilBlock:^BOOL(NSString *value) {
		return [value isEqualToString:@"xxx"];
     }] subscribeNext:^(NSString *value) {
		NSLog(@"current value is not `stop`: %@", value);
	 }];

    RAC(self.usernameTextField, backgroundColor) = [validUsernameSignal map:^id(NSNumber *passwordValid) {
		  return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
	 }];
    RAC(self.passwordTextField, backgroundColor) = [validPasswordSignal map:^id(NSNumber *passwordValid) {
		return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
	 }];
	
	RAC(self.signInButton,enabled) = [RACSignal combineLatest:@[validUsernameSignal,validPasswordSignal]
													  reduce:^id(NSNumber *usernameValid, NSNumber *passwordValid){
													  return @([usernameValid boolValue] && [passwordValid boolValue]);
													  }];
####RACScheduler Request
	__block NSInteger number = numberLimit;
	@weakify(self);
    RACSignal *timeSignal = [[[[[RACSignal interval:1.0f onScheduler:[RACScheduler mainThreadScheduler]] take:numberLimit] startWith:@(1)] map:^id(NSDate *date) {
        @strongify(self);
        if (number == 0) {
            return @YES;
        }
        else{
            number--;
            return @NO;
        }
    }] takeUntil:self.rac_willDeallocSignal];
    
    self.timeButton.rac_command = [[RACCommand alloc]initWithEnabled:timeSignal signalBlock:^RACSignal *(id input) {
        number = numberLimit;
        return timeSignal;
    }];
    
    NSURL *url = [NSURL URLWithString:@"xxx"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    
    [[[NSURLConnection rac_sendAsynchronousRequest:request] deliverOn:[RACScheduler mainThreadScheduler]] subscribeNext:^(id x) {
        // do refresh something
        [[[RACSignal interval:3 onScheduler:[RACScheduler mainThreadScheduler]] take:1] subscribeNext:^(id x) {
            NSLog(@"the result:%@",x)
        }];
    }];
											  
####RACDelegateProxy
    RACDelegateProxy *emailTextFeildproxy = [[RACDelegateProxy alloc] initWithProtocol:@protocol(UITextFieldDelegate)];
	[[emailTextFeildproxy rac_signalForSelector:@selector(textFieldDidEndEditing:)] subscribeNext:^(RACTuple *args) {
		UITextField *field  = [args first];
		[field resignFirstResponder];
		NSString *text = field.text;
		if (![Common isValiateEmail:text]) {
			[self showMessage:@"xxx"];
		}
	}];
	
	[[emailTextFeildproxy rac_signalForSelector:@selector(textFieldShouldReturn:)] subscribeNext:^(RACTuple *args) {
		UITextField *field  = [args first];
		[field resignFirstResponder];
	}];
	
	self.emailTextFeild.delegate = (id<UITextFieldDelegate>)emailTextFeildproxy;
	objc_setAssociatedObject(self.emailTextFeild, _cmd, emailTextFeildproxy, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
####Notification+Category
	[[[NSNotificationCenter defaultCenter] rac_addObserverForName:UITextFieldTextDidBeginEditingNotification object:self.usernameTextField] subscribeNext:^(id x) {
		self.usernameTextField.backgroundColor = [UIColor redColor];
	}];
####UIAlertView/UIActionSheet+Category
    UIAlertView *alertView = [UIAlertView alloc] initWithTitle:@"" message:@"" delegate:nil cancelButtonTitle:@"cancel" otherButtonTitles:@"other1",@"other2",nil];
    [alertView rac_buttonClickedSignal]subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
    [alertView show];
    
    UIActionSheet *actionSheet = [[UIActionSheet alloc] initWithTitle:@"" delegate:nil cancelButtonTitle:@"cancel"  destructiveButtonTitle:nil otherButtonTitles:@"other1",@"other2", nil];
	[[actionSheet rac_buttonClickedSignal]subscribeNext:^(id x) {
		NSLog(@"%@",x);
	}];
	[actionSheet showInView:self.view];
####UIGestureRecognizer+Category[UIGestureRecognizer-RACExtension](https://github.com/kaiinui/UIGestureRecognizer-RACExtension)
    UITapGestureRecognizer *tapGesture = [[UITapGestureRecognizer alloc] init];
	[self.view addGestureRecognizer:tapGesture];
	[[tapGesture rac_gestureSignal] subscribeNext:^(id x) {
		NSLog(@"%@",x);
	}];	

####UIRefreshControl+Category
    UIRefreshControl *refreshControl = [[UIRefreshControl alloc] init];
    [[refreshControl rac_signalForControlEvents:UIControlEventValueChanged] subscribeNext:^(id x) {
        NSLog("%@",x);
    }];
