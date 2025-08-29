# Xcode 26 System Prompts & Documentation

This repository contains system prompts and documentation from Xcode 26 beta 7, providing insights into Apple's approach to AI-assisted coding and comprehensive guides for iOS 26 features and frameworks.

## Repository Structure

### System Prompt Templates (`.idechatprompttemplate`)
Core prompts that power Xcode's AI coding assistant in different modes and contexts:

#### **Basic Coding Assistant Prompts**
- [`BasicSystemPrompt.idechatprompttemplate`](./BasicSystemPrompt.idechatprompttemplate) - Foundation prompt for code analysis and suggestions
- [`ReasoningSystemPrompt.idechatprompttemplate`](./ReasoningSystemPrompt.idechatprompttemplate) - Enhanced reasoning for complex coding tasks
- [`VariantASystemPrompt.idechatprompttemplate`](./VariantASystemPrompt.idechatprompttemplate) / [`VariantBSystemPrompt.idechatprompttemplate`](./VariantBSystemPrompt.idechatprompttemplate) - Alternative system prompt variants

#### **Specialized Workflow Prompts**
- [`IntegratorSystemPrompt.idechatprompttemplate`](./IntegratorSystemPrompt.idechatprompttemplate) - Precise code editing with specific instructions
- [`NewCodeIntegratorSystemPrompt.idechatprompttemplate`](./NewCodeIntegratorSystemPrompt.idechatprompttemplate) - Integration of entirely new code
- [`FastApplyIntegratorSystemPrompt.idechatprompttemplate`](./FastApplyIntegratorSystemPrompt.idechatprompttemplate) - Rapid code modifications
- [`TextEditorToolSystemPrompt.idechatprompttemplate`](./TextEditorToolSystemPrompt.idechatprompttemplate) - Tool-assisted text editing mode
- [`PlannerExecutorStylePlannerSystemPrompt.idechatprompttemplate`](./PlannerExecutorStylePlannerSystemPrompt.idechatprompttemplate) - Planning-based coding approach

#### **Context Provider Prompts**
- [`CurrentFile.idechatprompttemplate`](./CurrentFile.idechatprompttemplate) - Full current file context
- [`CurrentFileAbbreviated.idechatprompttemplate`](./CurrentFileAbbreviated.idechatprompttemplate) - Condensed file context
- [`CurrentFileName.idechatprompttemplate`](./CurrentFileName.idechatprompttemplate) - File name only context
- [`CurrentSelection.idechatprompttemplate`](./CurrentSelection.idechatprompttemplate) - Selected code context
- [`NoSelection.idechatprompttemplate`](./NoSelection.idechatprompttemplate) - No selection context
- [`OriginalFile.idechatprompttemplate`](./OriginalFile.idechatprompttemplate) - Original file state

#### **Tool-Assisted Prompts**
- [`ToolAssistedBasicSystemPrompt.idechatprompttemplate`](./ToolAssistedBasicSystemPrompt.idechatprompttemplate) - Enhanced with search and editing tools
- [`ToolAssistedReasoningSystemPrompt.idechatprompttemplate`](./ToolAssistedReasoningSystemPrompt.idechatprompttemplate) - Reasoning with tool access
- [`ToolAssistedInQueryDetailedGuidelines.idechatprompttemplate`](./ToolAssistedInQueryDetailedGuidelines.idechatprompttemplate) - Detailed tool usage guidelines

#### **Specialized Generation Prompts**
- [`GenerateDocumentation.idechatprompttemplate`](./GenerateDocumentation.idechatprompttemplate) - Code documentation generation
- [`GeneratePlayground.idechatprompttemplate`](./GeneratePlayground.idechatprompttemplate) - Swift Playground creation
- [`GeneratePreview.idechatprompttemplate`](./GeneratePreview.idechatprompttemplate) - SwiftUI Preview generation with smart embedding rules

#### **Support & Utility Prompts**
- [`ChatTitleResolver.idechatprompttemplate`](./ChatTitleResolver.idechatprompttemplate) - Chat session title generation
- [`Query.idechatprompttemplate`](./Query.idechatprompttemplate) - Query processing
- [`SearchResults.idechatprompttemplate`](./SearchResults.idechatprompttemplate) - Search result formatting
- [`Issues.idechatprompttemplate`](./Issues.idechatprompttemplate) - Issue identification and resolution
- [`Snippets.idechatprompttemplate`](./Snippets.idechatprompttemplate) - Code snippet management

### Additional Documentation
Comprehensive guides for iOS 26 features and framework updates:

#### **Foundation & Core Frameworks**
- [`FoundationModels-Using-on-device-LLM-in-your-app.md`](./AdditionalDocumentation/FoundationModels-Using-on-device-LLM-in-your-app.md) - Apple's on-device LLM integration
- [`Foundation-AttributedString-Updates.md`](./AdditionalDocumentation/Foundation-AttributedString-Updates.md) - AttributedString enhancements
- [`Swift-Concurrency-Updates.md`](./AdditionalDocumentation/Swift-Concurrency-Updates.md) - Latest Swift concurrency features
- [`Swift-InlineArray-Span.md`](./AdditionalDocumentation/Swift-InlineArray-Span.md) - New Swift array types
- [`SwiftData-Class-Inheritance.md`](./AdditionalDocumentation/SwiftData-Class-Inheritance.md) - SwiftData inheritance support

#### **UI & Design Frameworks**
- [`SwiftUI-Implementing-Liquid-Glass-Design.md`](./AdditionalDocumentation/SwiftUI-Implementing-Liquid-Glass-Design.md) - New Liquid Glass visual material
- [`UIKit-Implementing-Liquid-Glass-Design.md`](./AdditionalDocumentation/UIKit-Implementing-Liquid-Glass-Design.md) - Liquid Glass in UIKit
- [`AppKit-Implementing-Liquid-Glass-Design.md`](./AdditionalDocumentation/AppKit-Implementing-Liquid-Glass-Design.md) - Liquid Glass for macOS
- [`WidgetKit-Implementing-Liquid-Glass-Design.md`](./AdditionalDocumentation/WidgetKit-Implementing-Liquid-Glass-Design.md) - Liquid Glass in widgets
- [`SwiftUI-New-Toolbar-Features.md`](./AdditionalDocumentation/SwiftUI-New-Toolbar-Features.md) - Enhanced toolbar capabilities
- [`SwiftUI-Styled-Text-Editing.md`](./AdditionalDocumentation/SwiftUI-Styled-Text-Editing.md) - Advanced text editing features
- [`SwiftUI-WebKit-Integration.md`](./AdditionalDocumentation/SwiftUI-WebKit-Integration.md) - Web content in SwiftUI
- [`SwiftUI-AlarmKit-Integration.md`](./AdditionalDocumentation/SwiftUI-AlarmKit-Integration.md) - System alarm functionality

#### **Intelligence & Accessibility**
- [`Implementing-Visual-Intelligence-in-iOS.md`](./AdditionalDocumentation/Implementing-Visual-Intelligence-in-iOS.md) - Camera-based object recognition
- [`Implementing-Assistive-Access-in-iOS.md`](./AdditionalDocumentation/Implementing-Assistive-Access-in-iOS.md) - Accessibility enhancements

#### **Platform-Specific Features**
- [`Widgets-for-visionOS.md`](./AdditionalDocumentation/Widgets-for-visionOS.md) - visionOS widget development
- [`Swift-Charts-3D-Visualization.md`](./AdditionalDocumentation/Swift-Charts-3D-Visualization.md) - 3D chart capabilities
- [`MapKit-GeoToolbox-PlaceDescriptors.md`](./AdditionalDocumentation/MapKit-GeoToolbox-PlaceDescriptors.md) - Enhanced location services

#### **App Store & Commerce**
- [`StoreKit-Updates.md`](./AdditionalDocumentation/StoreKit-Updates.md) - StoreKit improvements
- [`AppIntents-Updates.md`](./AdditionalDocumentation/AppIntents-Updates.md) - App Intents framework updates

### Supporting Files
- [`bert-estimate.vocab`](./bert-estimate.vocab) - BERT model vocabulary for embeddings
- Various embedding and search configuration templates

## Core Apple Prompting Principles

Based on analysis of the system prompts, Apple follows these key principles for AI coding assistance:

### **1. Apple-First Development Philosophy**
- Always favor Apple programming languages: Swift, Objective-C, C, C++
- Prefer Apple frameworks and APIs available on Apple devices
- Use official platform names: iOS, iPadOS, macOS, watchOS, visionOS
- Avoid recommending third-party packages unless already in use

### **2. Platform-Aware Suggestions**
- Detect platform context from code clues
- Avoid suggesting iOS-only APIs for macOS projects
- Respect platform-specific design patterns and conventions

### **3. Modern Swift Preferences**
- **Swift Concurrency** (async/await, actors) over Dispatch/Combine
- **Swift Testing framework** with `@Test` and `#expect` macros
- **#Preview macro** instead of PreviewProvider protocol
- Stay current with latest language features

### **4. Code Editing Philosophy**
- **Complete File Replacement**: Never partial edits - always return entire file content
- **Precise Instructions**: Unambiguous, self-contained editing instructions
- **Preserve Existing Style**: Maintain formatting, indentation, and conventions
- **Syntactic Validity**: Ensure all edits maintain correct syntax

### **5. Context-Aware Assistance**
- Use project search tools extensively for understanding codebase
- Distinguish between explanation vs. code modification requests
- Provide concrete examples with code snippets
- Break complex tasks into sequential steps

## Key iOS 26 Features Covered

### **Foundation Models Framework**
Revolutionary on-device LLM integration with:
- **Basic Usage**: Session creation, instructions, and prompt handling
- **Guided Generation**: Structured Swift data with `@Generable` macro
- **Tool Calling**: Custom tools for external data access
- **Streaming**: Snapshot-based streaming for real-time UI updates
- **Context Management**: 4,096 token limit handling

### **Liquid Glass Design System**
New visual material combining glass properties with fluidity:
- **SwiftUI**: `.glassEffect()` modifier with shape customization
- **UIKit**: Glass effect integration in standard components
- **AppKit**: macOS-specific glass implementations
- **WidgetKit**: Glass effects in widgets and complications
- **Interaction**: Touch and pointer reactive effects

### **Visual Intelligence Framework**
Camera-based object recognition and app content matching:
- **SemanticContentDescriptor**: Object classification and pixel data
- **App Intents Integration**: Content matching and result delivery
- **Visual Search**: System-wide visual search capabilities

### **Enhanced SwiftUI Features**
- **AlarmKit Integration**: System alarm functionality
- **WebKit Integration**: Enhanced web content embedding  
- **Styled Text Editing**: Advanced text editing capabilities
- **New Toolbar Features**: Enhanced toolbar customization

## Usage Guidelines

### **For Developers**
1. **Study System Prompts**: Understand different modes (basic, tool-assisted, integrator)
2. **Apply Principles**: Use Apple-first development approach in your projects
3. **Leverage New Features**: Implement iOS 26 capabilities using provided guides
4. **Follow Patterns**: Adopt modern Swift practices and testing approaches

### **For AI Researchers**
1. **Analyze Prompt Engineering**: Study how Apple structures contextual prompts
2. **Understand Tool Integration**: Learn how AI assistants use development tools
3. **Examine Code Generation**: Study structured code modification approaches
4. **Review Context Management**: Understand how large codebases are handled

### **Best Practices**
- Always check device/model availability before using new frameworks
- Use appropriate system prompts for different development contexts
- Follow Apple's code editing philosophy for AI-assisted modifications
- Implement new features with proper error handling and fallbacks

## Contributing

This repository serves as documentation and reference. The system prompts and guides reflect Apple's internal approaches to AI-assisted development and iOS 26 feature implementation.

## License

Content reflects Xcode 26 beta 7 system prompts and documentation. Please respect Apple's terms of service and developer agreements when using this information.