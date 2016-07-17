# PopupWindowAction�ŏo����Window��Region���g�p����

�\��̒ʂ�̂��Ƃ����Ă���T���v���v���O�����ł��B

- MainWindow
    - ViewA
	    - ChildViewA(PopupWindow��Content)
		    - ChildContentViewA
			- ChildContentViewB

��View����`����Ă��܂��B

MainWindow��Shell�ŁA���̒��ɒ�`����Ă���Region��ViewA���\������Ă��܂��B�����āAPopupWindow��ChildViewA���\������āA���̒��Œ�`����Ă���Region��ChildContentViewA��ChildContentViewB����ʑJ�ڂ��܂��B

# PopupWindowAction�ŐV����RegionManager�𐶐�����

PopupWindowAction�ŕ\�������Window�i���m�ɂ́A���̃R���e���c�j�ɐV����RegionManager�����蓖�Ă邽�߂ɁA���̃T���v���v���O�����ł�Behavior���g�p���Ă��܂��B
�ȉ��̂悤��Behavior���`���Ă��܂��B

```cs
using Microsoft.Practices.ServiceLocation;
using Prism.Common;
using Prism.Regions;
using System;
using System.Windows;
using System.Windows.Interactivity;

namespace NewRegionManagerPopupWindowSampleApp.Commons
{
    public class CreateNewRegionManagerBehavior : Behavior<DependencyObject>
    {
        protected override void OnAttached()
        {
            var rm = ServiceLocator.Current.GetInstance<IRegionManager>();
            var newRegionManager = rm.CreateRegionManager();
            RegionManager.SetRegionManager(this.AssociatedObject, newRegionManager);
            Action<IRegionManagerAware> setRegionManager = x => x.RegionManager = newRegionManager;
            MvvmHelpers.ViewAndViewModelAction(this.AssociatedObject, setRegionManager);
        }

        protected override void OnDetaching()
        {
            RegionManager.SetRegionManager(this.AssociatedObject, null);
            Action<IRegionManagerAware> resetRegionManager = x => x.RegionManager = null;
            MvvmHelpers.ViewAndViewModelAction(this.AssociatedObject, resetRegionManager);
        }
    }
}
```

�O���[�o����ServiceLocator����RegionManager���擾���āA��������V����RegionManager���쐬���āABehavior��K�p�����I�u�W�F�N�g�ɐݒ肵�Ă��܂��B
IRegionManagerAware�C���^�[�t�F�[�X�́ARegionManager���Z�b�g�ł��邾���̃V���v���ȃC���^�[�t�F�[�X�ł��B

```cs
using Prism.Regions;

namespace NewRegionManagerPopupWindowSampleApp.Commons
{
    public interface IRegionManagerAware
    {
        IRegionManager RegionManager { get; set; }
    }
}
```

# Region�̉�ʑJ�ڎ��ɁA���̎��g�p����Ă���RegionManager��ViewModel�ɐݒ肷��d�g��
DI�R���e�i����IRegionManager��ݒ肷��f�t�H���g�̕��@���ƁA��Ƀ��[�g��RegionManager���������܂�Ă��܂��܂��B
��LBehavior�ō쐬���ꂽRegionManager���������ނ��߂Ɉȉ��̂悤��Prism��RegionBehavior�Ƃ����d�g�݂��g����RegionManager��ViewModel�ɍ������ނ悤�ɂ��܂����B

```cs
using Prism.Common;
using Prism.Regions;
using System;
using System.Collections.Specialized;

namespace NewRegionManagerPopupWindowSampleApp.Commons
{
    public class RegionManagerAwareBehavior : RegionBehavior
    {
        public static string Key { get; } = nameof(RegionManagerAwareBehavior);

        protected override void OnAttach()
        {
            this.Region.ActiveViews.CollectionChanged += this.ActiveViews_CollectionChanged;
        }

        private void ActiveViews_CollectionChanged(object sender, System.Collections.Specialized.NotifyCollectionChangedEventArgs e)
        {
            switch (e.Action)
            {
                case NotifyCollectionChangedAction.Add:
                    Action<IRegionManagerAware> setRegionManager = x => x.RegionManager = this.Region.RegionManager;
                    MvvmHelpers.ViewAndViewModelAction(e.NewItems[0], setRegionManager);
                    break;
                case NotifyCollectionChangedAction.Remove:
                    Action<IRegionManagerAware> resetRegionManager = x => x.RegionManager = null;
                    MvvmHelpers.ViewAndViewModelAction(e.OldItems[0], resetRegionManager);
                    break;
            }
        }
    }
}
```

View���ǉ����ꂽ��IRegionManagerAware���ݒ肳��Ă���RegionManager��ݒ肷��Ƃ��������̃V���v���Ȃ������ł��B
�����Bootstrapper��ConfigureDefaultRegionBehaviors���I�[�o�[���C�h���ēo�^���Ă����܂��B

```cs
protected override IRegionBehaviorFactory ConfigureDefaultRegionBehaviors()
{
    var f = base.ConfigureDefaultRegionBehaviors();
    f.AddIfMissing(RegionManagerAwareBehavior.Key, typeof(RegionManagerAwareBehavior));
    return f;
}
```

# �g������

ChildViewA��ViewModel���Ɏg������������܂��BIRegionManagerAware���������邱�ƂŁA���݂�View�ɐݒ肳��Ă���RegionManager���ݒ肳���悤�ɂȂ�܂��B
���Ƃ́A������g���ĉ�ʑJ�ڂ��s�������ł��B

```cs
using NewRegionManagerPopupWindowSampleApp.Commons;
using Prism.Mvvm;
using Prism.Regions;

namespace NewRegionManagerPopupWindowSampleApp.ViewModels
{
    public class ChildViewAViewModel : BindableBase, IRegionManagerAware
    {
        private IRegionManager regionManager;

        public IRegionManager RegionManager
        {
            get { return this.regionManager; }
            set { this.SetProperty(ref this.regionManager, value); }
        }

        public void Initialize()
        {
            this.RegionManager.RequestNavigate("Main", "ChildContentViewA");
        }
    }
}
```

���ƁA�V����RegionManager���쐬������View�i����̏ꍇ��ChildViewA�j��CreateNewRegionManagerBehavior��ݒ肵�Ă������ƂŁA�����I��RegionManager���쐬����܂��B

```xml
<UserControl xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:local="clr-namespace:NewRegionManagerPopupWindowSampleApp.Views"
             xmlns:Prism="http://prismlibrary.com/"
             xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"
             xmlns:Commons="clr-namespace:NewRegionManagerPopupWindowSampleApp.Commons"
             xmlns:ei="http://schemas.microsoft.com/expression/2010/interactions"
             x:Class="NewRegionManagerPopupWindowSampleApp.Views.ChildViewA"
             Prism:ViewModelLocator.AutoWireViewModel="True"
             mc:Ignorable="d"
             d:DesignHeight="300"
             d:DesignWidth="300">
    <i:Interaction.Triggers>
        <i:EventTrigger>
            <ei:CallMethodAction TargetObject="{Binding}"
                                 MethodName="Initialize" />
        </i:EventTrigger>
    </i:Interaction.Triggers>
    <i:Interaction.Behaviors>
        <Commons:CreateNewRegionManagerBehavior />
    </i:Interaction.Behaviors>
    <Grid>
        <ContentControl Prism:RegionManager.RegionName="Main" />
    </Grid>
</UserControl>
```

ChildViewA���Œ�`����Ă���Region�ł̉�ʑJ�ڂ́AChildViewA��Loaded�C�x���g��ViewModel��Initialize���\�b�h���ĂԂ��ƂŎ������Ă��܂��B
