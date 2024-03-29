---
layout: post
title: Page Overlays:An Enhanced Virtual Memory Framework to Enable Fine-grained Memory Management
date: 2021-12-15
category: translation
---

# 摘要

许多最近的工作提出了一种机制来证明在细(例如，高速缓存行)粒度上管理内存的潜在优势，如细粒度重复数据删除和细粒度内存保护。不幸的是，现有的虚拟内存系统在更大的粒度上跟踪内存(例如，4 KB页面)，阻碍了此类技术的有效实现。简单地减少页大小会导致页表开销和TLB压力的不可接受的增加。

我们提出了一个新的虚拟内存框架，它能够有效地实现各种细粒度的内存管理技术。在我们的框架中，除了常规的物理页面外，每个虚拟页面都可以映射到一个称为page overlay的结构。overlay包含来自虚拟页面的缓存行的子集。存在于overlay中的缓存行从那里访问，所有其他缓存行从常规物理页访问。我们的page overlay框架支持缓存行粒度的内存管理，而不会显著改变现有的虚拟内存框架或引入高开销。

我们展示了我们的框架可以实现七种内存管理技术的简单而有效的实现，每种技术都有广泛的应用程序。我们定量地评估了这两种技术的潜在好处:overlay-on-write和稀疏数据结构计算。我们的评估表明，与传统的写时复制相比，将overlay-on-write应用于fork可以提高15%的性能，并平均减少53%的内存容量需求。对于稀疏数据计算，我们的框架可以在许多真实世界的稀疏矩阵上胜过最先进的基于软件的稀疏表示。我们的框架是通用的、强大的、有效的，能够以低成本实现细粒度内存管理。

# 1. 引言

虚拟存储器[17,21,30]是计算机架构领域最重要的发明之一。除了支持一些核心操作系统功能，如内存容量管理、进程间保护和数据共享，虚拟内存还支持一些简单的技术实现，从而显著提高性能，例如。，写时复制[22]和分页[3,7]。由于其简单性和强大的功能，几乎所有现代高性能系统都使用虚拟内存。

尽管它取得了成功，但目前实现的虚拟内存以粗粒度(页面)跟踪内存的事实是一个主要缺点。跟踪内存页粒度1)在许多现有的技术带来了明显的低效率(例如,即写即拷),和2)难以实现一些曾经提出的技术，如重复数据删除，细粒度数据保护(58 59)，缓存行级别压缩[37]，元数据管理[35,60]。例如，考虑写时复制技术，其中包含相同数据的多个虚拟页被映射到单个物理页:当修改任何虚拟页中的单个字节时，操作系统创建整个物理页的副本。该操作不仅会浪费内存空间，而且会导致较高的延迟、带宽和电量消耗[43]。

以比页面更细的粒度管理内存可以实现几种技术，这些技术可以显著提高系统性能和效率。然而，仅仅减少页面大小就会导致虚拟到物理映射表开销和TLB压力的不可接受的增加。以前解决此问题的工作要么依赖软件技术[23]（高性能开销），要么针对特定应用提出硬件支持[35、45、60]（低成本），要么显著修改现有虚拟内存的结构（采用成本高）。 

我们提出这样一个问题：我们能否构建一个通用的框架，在不显著改变现有虚拟内存框架的情况下，支持多种细粒度管理技术？作为回应，我们提出了一个新的虚拟内存（VM）框架，它通过一个称为page overlay的新概念来扩展现有的VM框架。 

![](/images/PageOverlays/Overview.png)

图1形象地比较了现有的虚拟内存框架和我们提出的框架。在现有框架中（图1a），流程的每个虚拟页面最多可以映射到一个物理页面。系统使用一组映射表来确定虚拟页面和物理页面之间的映射。在我们提出的框架中（图1b），每个虚拟页面还可以映射到一个称为overlay的结构。在较高级别上，虚拟页的overlay仅包含来自虚拟页的缓存行的子集。当虚拟页面同时具有物理页面和overlay映射时，访问语义如下：overlay中存在的任何缓存行都从那里访问；所有其他缓存行都是从物理页访问的。 

我们的新框架具有简单的访问语义，和以前的方法相比，它有两个主要优点，可以实现细粒度内存管理。

![](/images/PageOverlays/summary_of_techniques_benefited.png)

首先，我们的框架是通用和强大的。它无缝地实现了各种技术的有效实施，以提高整体系统性能、降低内存容量需求并增强系统安全性。例如，在写时拷贝技术中，当映射到单个物理页的其中一个虚拟页接收到写操作时，我们的机制不创建物理页的完整副本，而只是使用修改的缓存行为虚拟页创建overlay。我们将这种机制称为写时覆盖。与写时复制相比，overlay写入1）显著减少了内存中的冗余量，2）从关键执行路径中删除了复制操作。总之，我们描述了七种不同的技术，它们可以在我们的框架之上有效地实现。表1总结了这些技术，说明了我们的框架相对于先前提出的机制的好处。第5节更详细地描述了这些技术。 

我们的框架的第二个主要优点是，它在很大程度上保留了现有虚拟内存框架的结构。这一点非常重要，因为它不仅可以实现我们框架的低开销实现，还可以使系统将overlay视为一种廉价的功能，可以打开，或者取决于特定系统从overlay中获益的程度（向后兼容性）。 

实现我们框架的语义向我们提出了许多设计问题，例如，如何以及何时创建overlay，overlay内的缓存行如何定位，overlay如何紧凑地存储在主内存中？我们发现，对这些问题的天真回答会导致显著的性能开销。在这项工作中，我们进行了几项观察，结果是我们的框架设计简单且开销低。第3节讨论了天真方法的缺点，并概述了我们提出的设计。第4节详细描述了我们的最终设计和相关的实施成本。

我们使用七种技术中的两种来定量评估框架的好处：1）写时覆盖，2）稀疏数据结构计算的有效机制。我们的评估表明，与传统的写上拷贝机制相比，写时覆盖（应用于fork[1]）将性能提高了15%，并将内存容量需求降低了53%。对于稀疏矩阵向量乘法，我们的框架始终优于基线密集表示法，甚至可以在[16]中三分之一以上的大型真实稀疏矩阵上优于CSR[26]，CSR是一种基于最先进软件的稀疏表示法。更重要的是，与许多软件表示不同，我们的实现允许动态更新稀疏数据结构，这对于基于软件的表示通常是一个代价高昂且复杂的操作。第5节更详细地讨论了这些结果。

综上所述，本文做出了以下贡献。 
- 我们提出了一个新的虚拟内存框架，它通过一个称为页面overlay的新概念来扩展现有的VM框架。
- 我们展示了我们的框架提供了强大的访问语义，无缝地支持七种细粒度内存管理技术的高效实现。
- 我们详细介绍了我们提出的框架的一个简单、低开销的设计。我们的设计在常规内存访问的关键路径上添加了可忽略不计的逻辑。
- 我们使用两种细粒度的内存管理技术对我们的框架进行了定量评估，结果表明它显著提高了系统的整体性能并减少了内存消耗。

# 2. 动机

我们首先对我们提出的虚拟内存框架的语义进行了详细的概述。虽然我们提出的框架能够有效地实现表1中的每一种技术，但我们将使用一种这样的技术——写时覆盖来说明我们框架的威力。 

## 2.1 框架语义概述

如图1b所示，在我们提出的框架中，每个虚拟页面可以映射到两个实体：一个常规物理页面和一个页面overlay。页面overlay有两个方面。首先，与物理页（其大小与虚拟页相同）不同，虚拟页的overlay仅包含来自该页的缓存行的子集，因此其大小小于虚拟页。第二，当虚拟页面同时具有物理页面和overlay映射时，我们定义访问语义，以便从那里访问overlay中存在的任何缓存行。只有overlay中不存在的缓存行才能从物理页面访问，如图2所示，对于具有四条缓存行的页面的简化情况。在图中，虚拟页映射到物理页和overlay，但overlay仅包含缓存行C1和C3。根据我们的语义，对C1和C3的访问被映射到overlay，其余的缓存行被映射到物理页面。 

![](/images/PageOverlays/framework_semantics.png)

## 2.2 写时覆盖：更有效的写时复制 

写时复制是许多不同应用中广泛使用的原语，包括进程fork[1]、虚拟机克隆[32]、内存重复数据消除[55]、操作系统推测[10,36,57]和内存检查点[18,49,56]。图3a显示了写时拷贝技术的工作原理。最初，包含相同数据的两个（或多个）虚拟页映射到相同的物理页（P）。当其中一个页面收到写操作（导致数据发散）时，操作系统分两步处理。它首先识别一个新的物理页面（P'），并将原始物理页面（P）的内容复制到新页面。其次，它将接收写操作（V2）的虚拟页面重新映射到新的物理页面。这种重新映射操作通常需要TLB shootdown。

![](/images/PageOverlays/copy_on_write_vs_overlay_on_write.png)

尽管其应用广泛，但基于两个原因，即写即复制成本高昂，效率低下。首先，复制操作和重新映射操作都位于写操作的关键路径上。这两种操作都会导致高延迟。复制操作消耗高内存带宽，可能会降低共享内存带宽的其他应用程序的性能。其次，即使虚拟页面中只修改了一条缓存行，系统也需要创建整个物理页面的完整副本，从而导致内存容量的无效使用。 

我们的框架支持更快、更有效的机制（如图3b所示）。当多个虚拟页共享同一物理页时，操作系统会通过页表向硬件明确指示应在写入时复制这些页。当其中一个页面收到写操作时，我们的框架首先会创建一个overlay，其中只包含修改后的缓存行。然后，它将overlay映射到接收写入的虚拟页面。我们将这种机制称为写时覆盖。写时覆盖比写时复制有许多好处。首先，它避免了在写操作之前复制整个物理页的需要，从而显著减少了关键执行路径上的延迟（以及相关的内存带宽和能量增加）。第二，它允许系统消除主存储器中存储的数据中的显著冗余，因为与写时拷贝的整页相比，只需要存储overlay行。最后，如第4.3.3节所述，我们的设计利用了一个事实，即只有一条缓存行从源物理页重新映射到overlay，从而显著减少了重新映射操作的延迟。 

如前所述，写下复制有着广泛的应用。写时覆盖是一种比写时复制更快、更有效的替代方法，可以显著地为所有这些应用程序带来好处。

## 2.3 overlay语义的好处

对于表1中的大多数技术，我们的框架比现有的VM框架有两个不同的好处。首先，我们的框架减少了系统必须完成的工作量，从而提高了系统性能。例如，在overlay写和稀疏数据结构（第5.2节）技术中，我们的框架减少了需要复制/访问的数据量。其次，我们的框架能够显著降低内存容量需求。每个overlay仅包含来自虚拟页面的缓存行子集，因此系统可以通过将overlay紧凑地存储在主内存中来减少总体内存消耗，即，对于每个overlay，仅存储overlay中实际存在的缓存行。我们在第5节中使用两种技术定量评估了这些好处，并表明我们的框架是有效的。 

# 3. Page Overlay：设计概述

虽然我们的框架强加了简单的访问语义，但要有效地实现所提出的语义，还有几个关键挑战。在本节中，我们首先讨论这些挑战，并概述如何应对这些挑战。然后，我们将全面概述我们提出的解决这些挑战的机制，从而实现框架的简单、高效和低开销设计。

## 3.1 实现Page Overlay的挑战

**挑战1**：检查缓存行是否是overlay的一部分。当处理器需要访问虚拟地址时，它必须首先检查所访问的缓存行是否是overlay的一部分。由于大多数现代处理器使用物理标记的一级缓存，因此此检查位于一级访问的关键路径上。为了解决这个问题，我们将每个虚拟页与一个位向量相关联，该位向量表示来自虚拟页的哪些缓存行是overlay的一部分。我们称这个位向量为overlay位向量（OBitVector）。我们将OBitVector缓存在处理器TLB中，从而使处理器能够快速检查访问的缓存行是否是overlay的一部分。

**挑战2**：识别overlay缓存行的物理地址。如果访问的缓存行是overlay的一部分（即，它是overlay缓存行），处理器必须快速确定overlay缓存行的物理地址，因为访问一级缓存需要该地址。解决此问题的简单方法是将overlay存储在主内存中的区域的基址存储在TLB中（我们将此区域称为overlay存储）。虽然这可能使处理器能够识别具有唯一物理地址的每个overlay缓存行，但当overlay存储在主内存中时，这种方法有三个缺点。
- 首先，overlay存储（在主存中）不包含来自虚拟页的所有缓存行。因此，处理器必须执行一些计算来确定访问的overlay缓存行的地址。这将延迟L1缓存访问。
- 其次，大多数现代处理器使用虚拟索引的物理标记一级缓存来部分重叠一级缓存访问和TLB访问。这种技术要求缓存行的虚拟索引和物理索引相同。但是，由于overlay比虚拟页小，缓存行的overlay物理索引可能与缓存行的虚拟索引不同。因此，必须延迟缓存访问，直到TLB访问完成。
- 最后，将新缓存行插入到overlay中是一个相对复杂的操作。根据overlay在主内存中的表示方式，将新缓存行插入overlay可能会更改overlay中其他缓存行的地址。处理这种情况可能需要一种复杂的机制来确保这些其他缓存行的一致性。

在我们的设计中，我们通过为每个overlay使用两个不同的地址来解决这一难题：一个用于寻址处理器缓存（称为overlay地址），另一个用于寻址主内存（称为overlay内存存储地址）。正如我们将很快描述的，这种双地址设计使系统能够独立于处理器缓存中如何寻址overlay缓存行来管理主内存中的overlay，从而克服上述三个缺点。

**挑战3**：确保TLB的一致性。在我们的设计中，由于TLB缓存OBitVector，当缓存行从物理页移动到overlay或从物理页移动到overlay时，任何缓存了对应虚拟页的映射的TLB都应该更新其映射以引用缓存行重新映射。解决这一挑战的天真方法是使用TLB击落，这很昂贵。幸运的是，在上述场景中，TLB映射只针对单个缓存行（而不是整个虚拟页面）进行更新。我们提出了一种利用这一事实的简单机制，并使用缓存一致性协议来保持TLB的一致性（第4.3.3节）。

## 3.2 设计概述

上面提到的双地址设计的一个关键方面是，访问缓存的地址（overlay地址）取自地址空间，其中每个overlay的大小与常规物理页面的大小相同。这使得我们的设计能够无缝地解决挑战2（overlay缓存行地址计算），而不会产生解决问题的天真方法的缺点（如第3.1节所述）。问题是，overlay地址是从哪个地址空间获取的？ 

为了回答这个问题，我们观察到只有一小部分物理地址空间是由主内存（DRAM）支持的，并且大部分物理地址空间是未使用的，即使在一部分被内存映射I/O和其他系统构造占用之后也是如此。我们建议将此未使用的物理地址空间用于overlay缓存地址，并将此空间称为overlay地址空间。

![](/images/PageOverlays/Overview_design.png)

图4显示了我们的设计概述。有三个地址空间：虚拟地址空间、物理地址空间和主存地址空间。主内存地址空间在常规物理页和overlay内存存储（OMS）之间分割，overlay内存存储（OMS）是一个紧凑存储overlay的区域。在我们的设计中，为了将虚拟页面与overlay相关联，首先使用直接映射将虚拟页面映射到overlay地址空间中的全尺寸页面，而不进行任何转换或间接寻址（第4.1节）。overlay页依次使用存储在内存控制器中的映射表映射到OMS中的某个位置（第4.2节）。我们将在第4节中更详细地描述该功能。 

## 3.3 设计的好处

我们的高级设计有三大好处。首先，我们的方法没有改变现有VM框架将虚拟页面映射到物理页面的方式。这一点非常重要，因为系统可以将overlay视为一种廉价的功能，只有当应用程序从中受益时才需要启用。其次，如前所述，通过为每个overlay使用两个不同的地址，我们的实现将缓存寻址方式与overlay存储在主内存中的方式解耦。这使得系统能够以非常类似于常规缓存访问的方式处理overlay缓存访问，因此只需要对现有硬件结构进行很少的更改（例如，它可以无缝地处理虚拟索引的物理标记缓存）。第三，正如我们将在下一节中描述的那样，在我们的设计中，只有当缓存层次结构中的访问完全丢失时，才会访问overlay内存存储（在主存中）。这1）大大减少了与管理OMS相关的操作数量；2）减少了需要缓存在处理器TLB中的信息量；3）更重要的是，使内存控制器能够在与操作系统交互最少的情况下完全管理OMS。

# 4. Page Overlay：详细设计

概括一下我们的高级设计（图4），系统中的每个虚拟页都映射到两个实体：1）一个常规物理页，它直接映射到主存中的一个页；2）overlay地址空间中的overlay页（不直接由主存支持）。该空间中的每个页面依次映射到overlay内存存储中的一个区域，overlay存储在该区域中。因为我们的实现没有修改虚拟页面映射到常规物理页面的方式，所以我们现在将注意力集中在虚拟页面如何映射到overlay上。

## 4.1 虚拟到overlay映射

虚拟到overlay映射将虚拟页面映射到overlay地址空间中的页面。维护此映射信息的一种简单方法是将其存储在页面表中，并允许操作系统管理映射（类似于常规物理页面）。但是，这会增加映射表的开销，并使操作系统复杂化。我们进行了一个简单的观察，并施加了一个约束，使虚拟到overlay映射成为直接1-1映射。

我们的观察结果是，由于overlay地址空间是未使用的物理地址空间的一部分，因此它可以明显大于主内存的数量。为了实现虚拟页面和overlay页面之间的1-1映射，我们施加了一个简单的约束，其中没有两个虚拟页面可以映射到同一个overlay页面。 

![](/images/PageOverlays/virtual-to-overlay_mapping.png)

图5显示了我们的设计如何将虚拟地址映射到相应的overlay地址。我们的方案拓宽了物理地址空间，使得通过简单地连接overlay位（设置为1）、PID和vaddr，可以获得与ID=PID的进程的虚拟地址vaddr相对应的overlay地址。由于两个虚拟页不能共享一个overlay，当一个虚拟页的数据复制到另一个虚拟页时，源页的overlay缓存行必须复制到目标页中的适当位置。虽然这种方法需要比现有系统稍宽的物理地址空间，但与将此映射显式存储在单独的表中相比，这是一种更实用的机制，这会导致比我们的方法更高的存储和管理开销。由于每个进程有64位物理地址空间和48位虚拟地址空间，这种方法可以支持2^15个不同的进程。

请注意，由于同义词问题，无法使用类似的方法将虚拟页映射到物理页，这是多个虚拟页映射到同一物理页的结果。但是，由于我们施加的约束，虚拟到overlay映射不会出现此问题：没有两个虚拟页面可以映射到同一overlay页面。即使有这种限制，我们的框架也支持许多可以提高性能和减少内存容量需求的应用程序（第5节）。 

## 4.2 overlay地址映射

在overlay地址空间中标记的overlay缓存行必须在逐出时映射到overlay内存存储位置。在我们的设计中，由于虚拟页面和overlay页面之间存在1-1映射，因此我们可以将该映射与物理页面映射一起存储在页面表中。但是，由于许多页面可能没有overlay，我们将此映射信息存储在一个单独的映射表中，类似于页面表。这个overlay映射表（OMT）完全由内存控制器维护和控制，与操作系统的交互最少。第4.4节详细描述了overlay内存存储管理。

## 4.3 微体系结构和内存访问操作

图6描述了我们设计的微体系结构细节。当前系统的微体系结构有三个主要变化。首先（图中①），主存分为两个区域，分别存储：1）常规物理页和2）overlay内存存储（OMS）。OMS存储overlay的压缩表示和overlay映射表（OMT），后者将每个页面从overlay地址空间映射到overlay内存存储中的某个位置。在较高级别上，每个OMT条目包含：1）OBitVector，表示overlay中是否存在对应页面中的每个缓存行；2）OMSaddr，表示overlay在OMS中的位置。第二②，我们用一个称为OMT缓存的缓存来扩充内存控制器，该缓存缓存最近从OMT访问的条目。第三③，由于TLB必须确定对虚拟地址的访问是否应定向到相应的overlay，因此我们扩展每个TLB条目以存储OBitVector。虽然这可能会增加每次TLB未命中的成本（因为它需要从OMT获取OBitVector），但我们的评估（第5节）表明，使用overlay的性能优势超过了额外TLB fll延迟的三分之一。

![](/images/PageOverlays/microarchitectural.png)

为了描述不同内存访问的操作，我们以写时覆盖（第2.2节）为例。假设两个虚拟页（V1和V2）在写时复制模式下映射到同一物理页，V2的一些缓存行已经映射到overlay。V2上有三种可能的操作：1）读取，2）写入overlay中已存在的缓存行（简单写入），以及3）写入overlay中不存在的缓存行（overlay写入）。现在，我们将详细描述这些操作中的每一项。

### 4.3.1 内存读取操作

当页面V2接收到读取请求时，处理器首先使用相应的页码（VPN）访问TLB，以检索物理映射（PPN）和OBitVector。它通过连接进程的地址空间ID（ASID）和VPN（如第4.1节所述）来生成overlay页码（OPN）。根据所访问的缓存行是否存在于overlay中（由OBitVector中的相应位指示），处理器使用PPN或OPN生成一级缓存标记。如果在整个缓存层次结构（L1到最后一级缓存）中访问未命中，则请求将发送到内存控制器。控制器通过检查物理地址中的overlay位来检查请求的地址是否是overlay地址空间的一部分。如果是这样，它将从OMT缓存中查找相应overlay页的overlay存储地址（OMSaddr），并计算主内存中请求的缓存行的确切位置（如第4.4节稍后所述）。然后，它从主内存访问缓存行，并将数据返回到缓存层次结构。 

### 4.3.2 简单写操作

当处理器接收到对overlay中已经存在的缓存行的写入时，它只需更新overlay中的缓存行。此操作的路径与读取操作的路径相同，只是缓存行在读入一级缓存后会更新。

### 4.3.3 overlay写操作

overlay写入操作是对overlay中尚未存在的缓存行的写入。由于虚拟页在写时复制模式下映射到常规物理页，因此必须将相应的缓存行重新映射到overlay（基于第2.2节中描述的语义）。我们分三步完成overlay写入：1）将常规物理页（PPN）中缓存行的数据复制到overlay地址空间页（OPN）中相应的缓存行，2）更新所有TLB和OMT以指示缓存行映射到overlay，以及3）处理写入操作。

第一步可以在硬件中完成，方法是从常规物理页读取缓存行，并简单地更新缓存标记以对应于overlay页码（或通过制作缓存行的显式副本）。天真地实现第二步将涉及对应虚拟页面的TLB shootdown。然而，我们利用三个简单的事实来使用缓存一致性网络来保持TLB和OMT的一致性：i）映射仅针对单个缓存行而不是整个页面进行修改，ii）overlay页面地址可用于唯一标识虚拟页面，因为虚拟页面之间没有共享overlay，以及iii）overlay地址是物理地址空间的一部分，因此也是缓存一致性网络的一部分。基于这些事实，我们提出了一种新的缓存一致性消息，称为overlay读独占。当核心接收到该请求时，它会检查其TLB是否缓存了虚拟页面的映射。如果是这样，内核只需在OBitVector中为相应的缓存行设置位。overlay读独占请求也被发送到内存控制器，以便它可以更新OMT中相应overlay页的OBitVector（通过OMT缓存）。重新映射操作完成后，写入操作（第三步）的处理与简单写入操作类似。

请注意，在overlay写入之后，相应的缓存行（我们称之为overlay缓存行）被标记为脏。但是，与写时复制（必须在写操作之前分配内存）不同，我们的机制在清除脏overlay缓存行时会延迟分配内存空间，这显著提高了性能。

### 4.3.4 将overlay转换为常规物理页面

根据使用overlay的技术，在某时间点之后可能不需要维护虚拟页面的overlay。例如，在使用写时覆盖时，如果虚拟页中的大多数缓存行都已修改，则在overlay中维护缓存行不会带来任何好处。系统可以采取三种操作之一将overlay升级到物理页面：**复制并提交**操作是指操作系统将数据从常规物理页面复制到新物理页面，并使用overlay中的相应数据更新新物理页面的数据。**提交**操作使用overlay中的相应数据更新常规物理页面的数据。**放弃**操作只是放弃overlay。

当复制并提交操作与overlay-on-write一起使用时，提交和放弃操作被使用，例如，在推测的上下文中，我们的机制将推测更新存储在overlay中（第5.3.3节）。在任何这些操作之后，系统将清除相应虚拟页的OBitVector，并释放为overlay分配的overlay内存存储空间（将在第4.4节中讨论）。

## 4.4 管理overlay内存存储

overlay内存存储（OMS）是主内存中存储所有overlay的区域。如第4.3节所述。1、仅当overlay访问在缓存层次结构中完全丢失时，才会访问OMS。因此，有许多简单的方法来管理OMS。一种方法是在内存控制器上安装一个小型嵌入式内核，该内核可以运行管理OMS的软件例程（现有系统支持类似的机制，例如Intel主动管理技术）。另一种方法是让内存控制器通过使用完整的物理页来存储每个overlay来管理OMS。虽然这种方法将放弃我们框架的内存容量优势，但它仍将获得减少总体工作量的优势（第2.3节）。

在本节中，我们将介绍一种硬件机制，该机制可以通过使用overlay减少工作和内存容量。在我们的机制中，内存控制器完全管理OMS，与操作系统的交互最少。管理OMS有两个关键方面。首先，由于每个overlay仅包含来自虚拟页面的缓存行子集，因此我们需要overlay的紧凑表示，以便OMS仅包含overlay中实际存在的缓存行。其次，内存控制器必须管理多个不同大小的overlay。我们需要一个简单的机制来处理这种不同的大小和相关的自由空间碎片问题。虽然分配新overlay或重新定位现有overlay的操作稍微复杂一些，但只有当脏overlay缓存行写回主存时才会触发这些操作。因此，这些操作很少，并且不在执行的关键路径上。

### 4.4.1 压缩overlay表示

紧凑地维护overlay的一种简单方法是按照缓存行在虚拟页面中的显示顺序将缓存行存储在overlay中。虽然此表示很简单，但如果在其他overlay缓存行之前将新缓存行插入overlay，则内存控制器必须移动这些缓存行，以便为插入的缓存行创建插槽。这是一个读-修改-写操作，会导致显著的性能开销。

我们提出了另一种机制，即在OMS中为每个overlay分配一个段。overlay与指针数组相关联，虚拟页中的每个缓存行对应一个指针。每个指针要么指向包含缓存行的overlay段内的插槽，要么在overlay中不存在缓存行时无效。我们将此元数据存储在段头的单个缓存行中。对于小于4KB大小的段，我们使用64个5位插槽指针和一个32位向量来指示352位段内的空闲插槽。对于4KB段，我们不存储任何元数据，只将每个overlay缓存行存储在offset中，offset与虚拟页面中缓存行的offset相同。图7显示了一个大小为256B的overlay段，其中只有虚拟页面的第一行和第四行缓存行映射到overlay。

![](/images/PageOverlays/overlay_segment.png)

### 4.4.2 管理多个overlay大小

不同的虚拟页面可能包含不同大小的overlay。内存控制器必须将它们有效地存储在可用空间中。为了简化这种管理，我们的机制将可用的overlay空间拆分为5个固定大小的段：256B、512B、1KB、2KB和4KB。每个overlay存储在足够大以存储overlay缓存行的最小段中。当内存控制器需要一个段用于新的overlay或当它想将现有overlay迁移到更大的段时，控制器会识别所需大小的空闲段，并用新段的基址更新相应overlay页的OMSaddr。单个缓存行在写入主内存时在段内分配其插槽。

### 4.4.3 空闲空间管理

为了管理overlay内存存储中的空闲段，我们使用了一种简单的基于链表的方法。对于每个段大小，内存控制器维护一个指向该大小的空闲段的内存位置或寄存器。每个空闲段依次存储指向相同大小的另一个空闲段的指针或表示列表结尾的无效指针。如果控制器用完了特定大小的自由段，它将获得下一个更大大小的自由段，并将其拆分为两个。如果控制器用完了可用的4KB段，它会向操作系统请求额外的一组4KB页面。在系统启动期间，操作系统会主动将一大块空闲页分配给内存控制器。为了减少管理空闲段所需的内存操作数量，我们使用了分组链表机制，类似于某些文件系统所使用的机制。

### 4.4.4 overlay映射表（OMT）和OMT缓存

OMT将页面从overlay地址空间映射到overlay内存存储中的特定段。对于overlay地址空间中的每个页面（即，对于每个OPN），OMT包含一个包含以下信息的条目：1）OBitVector，指示overlay中存在哪些缓存行；2）overlay内存存储地址（OMSaddr），指向存储overlay的段。为了降低OMT的存储成本，我们将其分层存储，类似于虚拟到物理映射表。内存控制器在寄存器中维护分层表的根地址。

OMT缓存存储有关最近访问的overlay的以下详细信息：OBitVector、OMSaddr和overlay段元数据（存储在段的开头）。要从overlay层访问缓存行，内存控制器使用overlay层页码（OPN）咨询OMT缓存。在命中的情况下，控制器获取必要的信息，以便使用overlay段元数据在overlay内存存储中定位缓存行。如果未命中，控制器将执行OMT遍历（类似于页表遍历）以查找相应的OMT条目，并将其插入OMT缓存中。它还读取overlay段元数据并将其缓存在OMT缓存条目中。控制器可以在overlay更新时修改OMT的条目。当这样一个修改的条目从OMT缓存中退出时，内存控制器更新内存中相应的OMT条目。

## 4.5 硬件成本和操作系统修改

在我们的设计中，硬件开销有三个来源：1）OMT缓存，2）更宽的TLB条目（用于存储OBitVector），3）更宽的缓存标记（由于更宽的物理地址空间）。每个OMT缓存项存储overlay页编号（48位）、overlay内存存储地址（48位）、overlay位向量（64位）、overlay页中每个缓存行的指针数组（64*5=320位）以及overlay段的空闲列表位向量（32位）。总共，每个条目消耗512位。因此，64条OMT缓存的大小为4KB。每个TLB条目都使用64位OBitVector进行扩展。在64个入口L1 TLB和1024个入口L2 TLB中，扩展TLB的总成本为8.5KB。最后，假设每个缓存标记项需要为每个标记增加16位，跨64KB的一级缓存、512KB的二级缓存和2MB的三级缓存，扩展缓存标记以容纳更宽的物理地址的成本为82KB。因此，总体硬件存储成本为94.5KB。请注意，通过限制虚拟地址空间中可以overlay的部分，可以减少每个标记项所需的附加位，从而降低总体硬件成本。

在我们的实现中，操作系统的主要变化与overlay内存存储管理有关。操作系统与内存控制器交互，在常规物理页和OMS之间动态划分可用的主内存空间。如第4.4节所述，只有当脏overlay缓存行写回主内存且内存控制器超出overlay内存存储空间时，才会触发此交互。此操作非常罕见，属于关键路径。此外，该操作可以在备用内核上运行，不必暂停任何正在运行的硬件线程。请注意，页面overlay的某些应用程序可能需要额外的操作系统或应用程序支持，如描述应用程序的章节（第5节）所述。

# 5. 应用及评估

![](/images/PageOverlays/parameter_simulated_system.png)

## 5.3 我们框架的其他应用

我们现在描述表1中的其他应用程序，这些应用程序可以在我们的框架之上高效地实现。虽然先前的工作已经为其中一些应用程序提出了机制，但我们的框架要么支持更简单的机制，要么支持先前工作提出的机制的有效硬件支持。由于篇幅不足，我们仅在较高层次上描述这些机制，并将更详细的解释推迟到未来的工作中。

### 5.3.1 细粒度重复数据消除

Gupta等人观察到，在使用同一客户操作系统运行多个虚拟机的系统中，有许多页面包含几乎相同的数据。他们的分析表明，利用这种冗余可以将内存容量需求减少50%。他们提出了Diference Engine，它使用小补丁在一个普通页面上存储类似的页面。但是，访问这些修补过的页面会带来很大的开销，因为操作系统必须在检索所需数据之前应用修补程序。我们的框架能够更有效地实现差分引擎，其中与基页不同的缓存行可以存储在overlay层中，从而实现对修补页的无缝访问，同时还降低了总体内存消耗。与HICAMP相比，HICAMP是一种缓存线级重复数据消除机制，它根据缓存线的内容定位缓存线，我们的框架避免了对现有虚拟内存框架和编程模型的重大更改。

### 5.3.2 高效检查点

检查点是高性能计算应用程序中的一个重要原语，在这种应用程序中，数据结构定期检查，以避免从一开始就重新启动长时间运行的应用程序。但是，检查点的频率和延迟通常受到需要写入备份存储的内存数据量的限制。通过我们的框架，可以使用覆盖来捕获两个检查点之间的所有更新。只有这些覆盖层需要写入备份存储以获取新的检查点，从而减少检查点的延迟和带宽。然后提交覆盖（第4.3.4节），以便每个检查点精确捕获自上一个检查点以来的增量。与之前关于有效检查点的工作（如INDRA、Reserve和Sheaved Memory）相比，我们的框架比INDRA和Reserve（与远程攻击恢复相关）更灵活，并且避免了Sheaved Memory的大量写放大（这会显著降低系统的整体性能）。

### 5.3.3 虚拟化推测

已经提出了几种基于硬件的推测技术（例如，线程级推测，事务内存），以提高系统性能。这些技术维护对缓存中内存的推测性更新。因此，当推测性更新的缓存线从缓存中移出时，这些技术必须声明推测不成功，从而导致潜在的机会浪费。在我们的框架中，这些技术可以在相应的覆盖中存储对虚拟页面的推测性更新。可以根据推测是否成功提交或放弃覆盖。这种方法不受缓存容量的限制，并支持潜在的无限推测。

### 5.3.4 细粒度元数据管理

存储有关数据的细粒度（例如，字粒度）元数据有多种应用（例如，memcheck、taintcheck、fnegrained protection、检测锁冲突）。先前的工作已经提出了有效存储和操作此类元数据的框架。然而，这些机制需要特定于存储和维护元数据的硬件支持。相比之下，通过我们的框架，系统可能会对每个虚拟页面使用覆盖来存储虚拟页面的元数据，而不是数据的替代版本。换句话说，覆盖地址空间用作虚拟地址空间的阴影内存。为了访问某些数据，应用程序使用常规的加载和存储指令。系统将需要新的元数据加载和元数据存储指令，以使应用程序能够从覆盖访问元数据。 

### 5.3.5 灵活的超级页面

许多现代体系结构支持超级页面，以减少TLB未命中的数量。事实上，最近的一项前期工作表明，单个任意大的超级页面（直接段）可以显著减少大型服务器的TLB未命中。不幸的是，使用超级页面降低了操作系统管理内存和实现写时拷贝等技术的灵活性。例如，据我们所知，没有一个系统在“写时拷贝”模式下跨两个进程共享超级页面。这种灵活性的缺乏带来了使用超级页面减少TLB未命中的好处和使用写时拷贝减少内存容量需求的好处之间的权衡。幸运的是，通过我们的框架，我们可以在更高级别的页面表条目上应用覆盖，使操作系统能够以更高的粒度管理超级页面。简言之，我们设想一种机制，将超级页面划分为更小的段（基于OBitVector中可用的位数），并允许系统潜在地将超级页面的段重新映射到覆盖。例如，当两个进程之间共享的超级页接收到写操作时，只复制相应的段，并设置OBitVector中的相应位。类似地，此方法可用于在超级页面中具有多个保护域。假设超级页面中只有少数段需要覆盖，这种方法仍然可以确保较低的TLB未命中率，同时为操作系统提供更高的灵活性。

# 6. 总结

我们引入了一个新的、简单的框架，它支持细粒度内存管理。我们的框架使用一个称为overlay的概念来扩充虚拟内存。每个虚拟页面都可以映射到物理页面和overlay。overlay仅包含来自虚拟页的缓存行的子集，overlay中存在的缓存行从虚拟页访问。我们展示了我们提出的框架，它具有简单的访问语义，支持几种细粒度的内存管理技术，同时不会显著改变现有的VM框架。我们通过两个应用程序定量地展示了我们的框架的好处：1）写时覆盖（一种有效的写时复制替代方案）和2）稀疏数据结构的有效硬件表示。我们的评估表明，我们的框架显著提高了性能，并降低了这两个应用程序的内存容量需求（例如，与传统的写时拷贝相比，平均性能提高15%，内存容量降低53%）。根据我们的结果，我们得出结论，我们提出的页面overlay框架是一种优雅而有效的方法，可以支持许多细粒度的内存管理应用程序。