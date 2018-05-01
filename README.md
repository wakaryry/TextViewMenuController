## How to use UIMenuController

```swift
var isUndoManagerEnabled: Bool = true {
        willSet {
            if self.isUndoManagerEnabled == newValue {
                return
            }
            
            self.undoManager?.levelsOfUndo = 10
            self.undoManager?.removeAllActions()
            self.undoManager?.setActionIsDiscardable(true)
        }
    }
    
//MARK: - Markdown Formatting
    /** true if the a markdown closure symbol should be added automatically after double spacebar tap, just like the native gesture to add a sentence period. Default is true.
     This will always be false if there isn't any registered formatting symbols.
     */
    var isFormattingEnabled: Bool {
        return ((self.registeredFormattingSymbols?.count ?? 0) > 0) ? true : false
    }
    
    // MARK: Menu MenuController
    /** An array of the registered formatting symbols. */
    var registeredSymbols: [String]? {
        return self.registeredFormattingSymbols
    }
    
    private var registeredFormattingTitles: [String]?
    private var registeredFormattingSymbols: [String]?
    private var isFormatting: Bool = false
    
    //MARK: - UIResponder Overrides
    override var canBecomeFirstResponder: Bool {
        self.myp_addCustomMenuControllerItems()
        return super.canBecomeFirstResponder
    }
    
    override func becomeFirstResponder() -> Bool {
        return super.becomeFirstResponder()
    }
    
    override var canResignFirstResponder: Bool {
        // Removes undo/redo items
        if self.isUndoManagerEnabled {
            self.undoManager?.removeAllActions()
        }
        return super.canResignFirstResponder
    }
    
    override func resignFirstResponder() -> Bool {
        return super.resignFirstResponder()
    }
    
    // MARK: - MenuController
    // need to change
    override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
        if self.isFormatting {
            let title = self.myp_formattingTitleFromSelector(action)
            let symbol = self.myp_formattingSymbolWithTitle(title)
            
            if symbol.count > 0 {
                if let x = self.delegate {
                    if x.responds(to: #selector(MYPTextViewDelegate.textView(_:shouldOfferFormattingForSymbol:))) {
                        return ((x as? MYPTextViewDelegate)?.textView(self, shouldOfferFormattingForSymbol: symbol))!
                    }
                }
                return true
            }
            return false
        }
        if action == #selector(delete(_:)) {
            return false
        }
        if action == #selector(myp_presentFormattingMenu(_:)) {
            return self.selectedRange.length > 0 ? true : false
        }
        if action == #selector(paste(_:)) && self.myp_isPasteboardItemSupported() {
            return true
        }
        
        if self.isUndoManagerEnabled {
            if action == #selector(myp_undo(_:)) {
                if self.undoManager!.undoActionIsDiscardable {
                    return false
                }
                return self.undoManager!.canUndo
            }
            if action == #selector(myp_redo(_:)) {
                if self.undoManager!.redoActionIsDiscardable {
                    return false
                }
                return self.undoManager!.canRedo
            }
        }
        
        return super.canPerformAction(action, withSender: sender)
    }
    
    override func paste(_ sender: Any?) {
        let pastedItem = self.myp_pastedItem()
        
        if pastedItem is [String : Any] {
            NotificationCenter.default.post(name: Notification.Name.MYPTextInputTask.MYPTextViewDidPasteItemNotification, object: nil, userInfo: pastedItem as? [AnyHashable : Any])
        }
        else if pastedItem is String {
            if let x = self.delegate {
                if x.responds(to: #selector(UITextViewDelegate.textView(_:shouldChangeTextIn:replacementText:))) {
                    if !x.textView!(self, shouldChangeTextIn: self.selectedRange, replacementText: pastedItem as! String) {
                        return
                    }
                }
            }
            // Inserting the text fixes a UITextView bug whitch automatically scrolls to the bottom
            // and beyond scroll content size sometimes when the text is too long
            self.myp_insertTextAtCaretRange(text: pastedItem as! String)
        }
    }
    
private func myp_addCustomMenuControllerItems() {
        let undo = UIMenuItem(title: NSLocalizedString("undo", comment: "撤回") , action: #selector(myp_undo(_:)))
        let redo = UIMenuItem(title: NSLocalizedString("redo", comment: "重做"), action: #selector(myp_redo(_:)))
        
        var items = [undo, redo]
        
        if self.registeredFormattingTitles?.count ?? 0 > 0 {
            let format = UIMenuItem(title: NSLocalizedString("format", comment: "格式"), action: #selector(myp_presentFormattingMenu(_:)))
            items.append(format)
        }
        
        UIMenuController.shared.menuItems = items
    }
    
    // need to change
    private func myp_formattingTitleFromSelector(_ sel: Selector) -> String {
        
        return ""
    }
    
    // need to change
    private func myp_formattingSymbolWithTitle(_ title: String) -> String {
        
        return ""
    }
    
    @objc private func myp_presentFormattingMenu(_ sender: Any) {
        var items = [UIMenuItem]()
        if let x = self.registeredFormattingTitles {
            for name in x {
                // how to solve this?
                let item = UIMenuItem(title: name, action: #selector(myp_fuck(_:)))
                items.append(item)
            }
        }
        self.isFormatting = true
        let menu = UIMenuController.shared
        menu.menuItems = items
        
        let manager = self.layoutManager
        let targetRect = manager.boundingRect(forGlyphRange: self.selectedRange, in: self.textContainer)
        
        menu.setTargetRect(targetRect, in: self)
        menu.setMenuVisible(true, animated: true)
    }
    
    @objc private func myp_undo(_ sender: Any) {
        self.undoManager?.undo()
    }
    
    @objc private func myp_redo(_ sender: Any) {
        self.undoManager?.redo()
    }
    
    // need to change
    @objc private func myp_fuck(_ sender: Any) {
        
    }
    
    // need to change
    private func myp_formart(_ title: String) {
        
    }
```
